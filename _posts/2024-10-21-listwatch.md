---
title: Kubernetes List-Watch机制（也没那么详的）详解
tags: 云控制系统
pageview: false
modify_date: 2024-10-13
aside:
  toc: true
---

<!--more-->

写了个list-watch的demo，至少看完了知道能怎么用。

首先是目录结构，大概包含了几个文件。

![alt text](/img/2024-10-21-listwatch/image.png)

主函数结构很简单，包括几个示例，可以按需取消注释。需要注意的是，`manager`是包名。

```go
func main() {
	util.InitConf()

	// test list API
	manager.List()

	// test watch API
	// manager.Watch()

	// test retrywatch API
	// manager.RetryWatch()

	// test informer API
	// manager.Inform()
}
```

首先，看一下`InitConf`函数内容，主要是根据读取配置文件里的`kubeconfig`，然后根据它建立客户端。于是就可以用客户端内置的各种API实现集群的控制。

```go
var (
	kubeconfig string
	KubeClient *kubernetes.Clientset
	Stopch     chan struct{} = make(chan struct{})
)

func InitConf() {
	flag.StringVar(&kubeconfig, "kubeconfig", "", "Path to a kubeconfig. Only required if out-of-cluster.")
	klog.InitFlags(nil)
	defer klog.Flush()
	flag.Parse()

	kubeconfig = "config.yaml"
	config, _ := clientcmd.BuildConfigFromFlags("", kubeconfig)
	var err error
	KubeClient, err = kubernetes.NewForConfig(config)
	if err != nil {
		klog.Exitf("Error building kubernetes clientset, error: %s", err.Error())
	}
}
```

首先看一下List的API。List API用于获取全量的资源数据变更。

```go
func List() {
	list, _ := util.KubeClient.CoreV1().Namespaces().List(context.Background(), v1.ListOptions{})

	for _, v := range list.Items {
		klog.Infof("get namespace %v", v.Name)
	}
}
```

然后看一下最原始的Watch的API。首先建立一个watcher结构体用于监听资源。这个watcher结构体里面包含了一个ResultChan，这个chan用来存储获取到的资源变更信息，变更信息的结构是watch.Event。变更信息里面包括消息结构体（比如这里的namespace）、结构体的变更情况（增删改）etc。根据增删改情形不同，可以指定不同的处理逻辑（可见`switch event.Type`部分）。

```go
func Watch() {
	// timeOut := int64(10)
	ticker := time.NewTicker(60 * time.Second)

	watcher, _ := util.KubeClient.CoreV1().Namespaces().Watch(
		context.Background(),
		metav1.ListOptions{}) // 一般不加这个

	for event := range watcher.ResultChan() {
		item := event.Object.(*corev1.Namespace)

		switch event.Type {
		case watch.Added:
			klog.Infof("add a new namespace %s", item.Name)
		case watch.Deleted:
			klog.Infof("delete a namespace %s", item.Name)
		case watch.Modified:
			klog.Infof("modify a namespace %s", item.Name)
		}

		select {
		case <-ticker.C:
			ticker.Stop()
			break
		default:
		}
	}

}
```

就这么使用watch有两个问题：
1. clientset 初始化完成后，利用 watch 接口是可以的，但是 APIServer 会定期关闭 watch 的连接，这样就会导致watch中断，还得自己实现重连：
    ![alt text](/img/2024-10-21-listwatch/image-1.png)
2. 你获取到的资源变更信息是一个变更序列，其区分方式是用结构体里的一个叫ResourceVersion的单调递增整数（数字越大，变更越新）。你有可能会对老版本信息做一个存储和维护机制（比如你要查老版本的东西），但是一帮人各自手搓存储机制又是在重复造轮子。所以希望官方能出一个存储机制。
3. 即使实现了存储机制，如果我监视了很多类型的资源，包括pod, service和自己定义的一堆crd，那我监控这些玩应都需要分别开一个watcher并且各开一个存储机制？这像话？是不是可以把这些watcher和内存都包好？
4. 如果没有缓存，那是不是获取资源都要访问apiserver？是不是会增大其负担？

于是，官方推荐使用 watch 的工具包。

首先看问题1，官方为了保障watch的连续性，包了一层重连机制并且交由Watcher托管，称为RetryWatcher。

```go
func RetryWatch() {

	watcher, _ := toolsWatch.NewRetryWatcher("1", &cache.ListWatch{WatchFunc: watcher})

	for event := range watcher.ResultChan() {
		item := event.Object.(*corev1.Namespace)

		switch event.Type {
		case watch.Added:
			klog.Infof("add a new namespace %s", item.Name)
		case watch.Deleted:
			klog.Infof("delete a namespace %s", item.Name)
		case watch.Modified:
			klog.Infof("modify a namespace %s", item.Name)
		}

		//select {
		//case <-ticker.C:
		//	ticker.Stop()
		//	break
		//default:
		//}
	}
}

// the watcher dies every 10s and the events will be replayed every time the watcher restarts.
func watcher(opts metav1.ListOptions) (watch.Interface, error) {
	timeOut := int64(10)
	return util.KubeClient.CoreV1().Namespaces().Watch(context.Background(), metav1.ListOptions{TimeoutSeconds: &timeOut})
}
```

最后，为解决问题2，我们用kubernetes的informer接口。

这个接口包装了常见的资源的watcher，以及一个队列。

所有的资源历史变更数据都被缓存在这个`sharedInformerF`结构体里了，获取资源的时候也是获取缓存里的数据。这样还可以减少对apiserver的访问。

执行流程如下：首先给资源注册回调函数，把满足指定条件的资源变更放在队列里。几个worker轮询队列里存放的变更，然后进行处理。

```go
type PodController struct {
	sharedInformerF informers.SharedInformerFactory
	Workqueue       workqueue.TypedRateLimitingInterface[string] // "namespace/name"

	PodLister listerv1.PodLister
}

func NewPodController() *PodController {
	pc := PodController{}
	pc.sharedInformerF = informers.NewSharedInformerFactory(util.KubeClient, 20*time.Second)
	// mark it as a var for further use
	podInformer := pc.sharedInformerF.Core().V1().Pods()
	podInformer.Informer().AddEventHandler(cache.ResourceEventHandlerFuncs{
		AddFunc: func(obj interface{}) {
			p := obj.(*v1.Pod)
			klog.Infof("Pod %s is added", p.Namespace+"/"+p.Name)
		},
		UpdateFunc: func(old, new interface{}) {
			o := old.(*v1.Pod)
			n := new.(*v1.Pod)
			if o.ResourceVersion != n.ResourceVersion {
				pc.Workqueue.Add(fmt.Sprintf("%s/%s", n.Namespace, n.Name))
			}
		},
		DeleteFunc: func(obj interface{}) {
			p := obj.(*v1.Pod)
			klog.Infof("Pod %s is deleted", p.Namespace+"/"+p.Name)
		},
	})
	pc.PodLister = podInformer.Lister()

	pc.Workqueue = workqueue.NewTypedRateLimitingQueue(workqueue.DefaultTypedControllerRateLimiter[string]())
	return &pc
}

func (pc *PodController) runWorker(ctx context.Context) {
	for pc.processNextWorkItem(ctx) {
	}
}

func (pc *PodController) processNextWorkItem(ctx context.Context) bool {
	podstr, shutdown := pc.Workqueue.Get()
	if shutdown {
		return false
	}
	defer pc.Workqueue.Done(podstr)

	err := func(key string) error {
		namespace, name, err := cache.SplitMetaNamespaceKey(key)
		if err != nil {
			return err
		}
        // REMINDER: data from cache!
		pod, err := pc.PodLister.Pods(namespace).Get(name)
		if err != nil {
			return err
		}
		klog.Infof("Pod %s/%s is modified", pod.Namespace, pod.Name)
		return nil
	}(podstr)

	if err != nil {
		pc.Workqueue.AddRateLimited(podstr)
		klog.Infof("%v", err)
	}
	return true
}

func (pc *PodController) Run(ctx context.Context) {
	defer utilruntime.HandleCrash()
	defer pc.Workqueue.ShutDown()

	pc.sharedInformerF.Start(util.Stopch)
	pc.sharedInformerF.WaitForCacheSync(util.Stopch)

	for i := 0; i < 1; i++ {
		go wait.UntilWithContext(ctx, pc.runWorker, time.Second)
		klog.Infof("add a runWorker")
	}
	<-ctx.Done()
}

func Inform() {
	ctx := context.Background()
	pc := NewPodController()
	pc.Run(ctx)

}

```