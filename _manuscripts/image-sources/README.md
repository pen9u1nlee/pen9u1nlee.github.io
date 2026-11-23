# 两篇自动驾驶笔记的图片整理记录

整理日期：2026-11-09。

本次逐张阅读两篇文章引用的 30 张截图，先梳理内容关系，再将文字、列表和重复信息转写为正文。保留 4 张确有展示作用的局部图，用程序裁剪、等比缩小并压缩为 WebP，没有重绘文字、曲线或箭头。

原始截图完整归档在本目录，移动前后已核对 SHA-256，文件索引见 [source-index.json](source-index.json)。本目录位于 `_manuscripts`，不作为正常博客页面或图片资源发布；原图仍存在于仓库中。各子目录的 `original-article.md` 保存改写前的文章快照，其中图片 URL 仍为当时的路径，仅用于回溯。

## 自动驾驶的技术演进与挑战

正文：[_posts/2024-07-23-AutoDriveTechChallenge.md](../../_posts/2024-07-23-AutoDriveTechChallenge.md)。

内容主线：

> 感知的数据驱动 → 多模块的数据驱动 → 端到端模型的拆分方式 → 数据完整性与复用性 → One-Model 的资源代价 → 当时选择 Two-Model 的原因 → 上汽案例。

| 原始截图 | 信息及前后关系 | 正文中的处理 |
| --- | --- | --- |
| [image-33.png](2024-07-23-AutoDriveTechChallenge/image-33.png) | 用特斯拉时间线引出技术演进 | 与 image-36 合并为三阶段表；不把硬件代际直接等同于软件架构 |
| [image-34.png](2024-07-23-AutoDriveTechChallenge/image-34.png) | 展开 V11 及之前的数据生产、感知、规划链条 | 转写为自动标注、仿真、数据引擎及规则规划的说明；原笔记的“HW2.0”改为材料所指的技术阶段 |
| [image-35.png](2024-07-23-AutoDriveTechChallenge/image-35.png) | 推测 V12 的共享特征和多解码器架构，配仪表盘画面 | 保留“推测”标记；用文字说明中间特征与可视化的区别，去掉不能由静态图证实的动态轨迹比较 |
| [image-36.png](2024-07-23-AutoDriveTechChallenge/image-36.png) | 将特斯拉个案扩展到整体技术路线，分出 Two-Model 与 One-Model | 与 image-33 合并；不沿用当时的未来年限预测 |
| [image-37.png](2024-07-23-AutoDriveTechChallenge/image-37.png) | 定义感知模型与决策规划模型，说明预测向规划扩展 | 转写为 Two-Model 的处理链和开发思路 |
| [image-38.png](2024-07-23-AutoDriveTechChallenge/image-38.png) | 列出模型输入输出，并提出传感器覆盖、导航缺失问题 | 输入输出合并成一张表；完整性问题单独成节 |
| [image-39.png](2024-07-23-AutoDriveTechChallenge/image-39.png) | 重复上一页输入输出，新增感知版本、车型差异问题 | 不重复抄表；增加数据复用性小节 |
| [image-40.png](2024-07-23-AutoDriveTechChallenge/image-40.png) | 用 One-Model 架构对照 Two-Model，并介绍算力代价 | 保留中间架构图；文字说明统一模型及训练成本，不将“完全无信息损失、全程无需标注”写成一般结论，不保留厂商算力规划数字 |
| [image-41.png](2024-07-23-AutoDriveTechChallenge/image-41.png) | 汇总两条路线的工程取舍 | 转成 Markdown 比较表，把路线选择限定在原分享的资源背景下 |
| [image-42.png](2024-07-23-AutoDriveTechChallenge/image-42.png) | 用上汽架构落实前面的 Two-Model 选择 | 裁剪保留数据连接图，配文字解释 BEV、PJP、自车、导航及地图信息 |
| [image-43.png](2024-07-23-AutoDriveTechChallenge/image-43.png) | 常规行车场景演示 | 与后两页合并为案例表：常规行车 → 换道 → 交互避让 |
| [image-44.png](2024-07-23-AutoDriveTechChallenge/image-44.png) | 静止或慢车前的主动换道演示 | 转为案例表中的行为说明 |
| [image-45.png](2024-07-23-AutoDriveTechChallenge/image-45.png) | 路口交互与横向避让演示 | 转为案例表中的行为说明；保留案例性质，不外推成功率 |

## 智能网联汽车云控多交通参与者测评系统

正文：[_posts/2024-07-21-CloudControlTestGen.md](../../_posts/2024-07-21-CloudControlTestGen.md)。

内容主线：

> 安全验证需求 → 稀有场景难复现、单次测试覆盖有限 → 多参与者、连续、主动的测试方式 → 云端与实体两类平台 → 场景生成 → 协同轨迹规划 → 考虑通信时延的执行控制 → 测试场演示。

| 原始截图 | 信息及前后关系 | 正文中的处理 |
| --- | --- | --- |
| [image.png](2024-07-21-CloudControlTestGen/image.png) | 产业发展与事故案例引出安全验证需求 | 概括动机，去掉不影响主线的市场规模和出货量预测；不保留截图无法证明的具体事故归因 |
| [image-1.png](2024-07-21-CloudControlTestGen/image-1.png) | 测试连接系统复杂性与安全、舒适、效率等需求 | 改写为首节的评价目标；去掉原笔记中与系统论证无关的 GPT 图灵测试断言 |
| [image-2.png](2024-07-21-CloudControlTestGen/image-2.png) | 稀疏度、长尾事件、交互难刻画共同引出加速测试 | 转为自然路测难以高效获取边缘场景的说明，不复述缺少推导条件的年限和里程估计 |
| [image-3.png](2024-07-21-CloudControlTestGen/image-3.png) | 固定条件下的刹停演示说明触发条件和适用场景问题 | 转为“单次未碰撞不等于完整可靠”的讨论 |
| [image-4.png](2024-07-21-CloudControlTestGen/image-4.png) | 将前面的例子归纳成五类测评不足 | 完整转为五行 Markdown 表格 |
| [image-5.png](2024-07-21-CloudControlTestGen/image-5.png) | 方案章节过渡标题 | 标题内容已经进入正文，去掉纯标题截图 |
| [image-6.png](2024-07-21-CloudControlTestGen/image-6.png) | 5G 云控、多参与者协同和实物全景 | 文字介绍方案，裁剪保留实物照片；不重复“国际首创”宣传语 |
| [image-7.png](2024-07-21-CloudControlTestGen/image-7.png) | 参与者从单一到混合 | 与 image-8、image-9 合成“谁参与、怎样发生、冲突来源”的比较表 |
| [image-8.png](2024-07-21-CloudControlTestGen/image-8.png) | 测试场景从割裂到连续 | 转写为连续动态测试的说明 |
| [image-9.png](2024-07-21-CloudControlTestGen/image-9.png) | 冲突生成从被动到主动 | 转写为封闭测试中的主动交互构造 |
| [image-10.png](2024-07-21-CloudControlTestGen/image-10.png) | 总览：一套系统、两类平台、三项技术 | 以平台职责表和顺序列表重建架构 |
| [image-11.png](2024-07-21-CloudControlTestGen/image-11.png) | 云端数字孪生、协议解析、群集调度、实时监控 | 纳入云端平台职责；控制台截图不再重复保留 |
| [image-12.png](2024-07-21-CloudControlTestGen/image-12.png) | 模拟目标的底盘、云控、感知特征和抗碰撞能力 | 转为实体平台能力说明；不保留缺少比较口径的成本比例 |
| [image-13.png](2024-07-21-CloudControlTestGen/image-13.png) | 场景库、TD3、高层决策与纵横向执行 | 转写为“边缘场景学习”小节，补回原文缺失的技术内容 |
| [image-14.png](2024-07-21-CloudControlTestGen/image-14.png) | 时空冲突关系与多车轨迹滚动优化 | 保留路口交会几何局部图；正文说明其作用，不抄符号定义不完整的公式 |
| [image-15.png](2024-07-21-CloudControlTestGen/image-15.png) | 通信时延模型、修正卡尔曼状态估计、横纵向控制 | 转成三步说明，替换原文“做一些修正”；不复述缺少单位和完整上下文的拟合参数 |
| [image-16.png](2024-07-21-CloudControlTestGen/image-16.png) | 连续动态城市 NOA 封闭测试场演示 | 转为系统应用说明，不重复“全球首家”宣传语 |

## 配图处理参数与体积

裁剪框采用原图像素坐标 `(left, top, right, bottom)`，右、下边界不包含在裁剪结果中。缩小使用 Lanczos，WebP 编码使用 method=6。原图保留不变。

| 原图 | 发布文件 | 裁剪框 | 输出尺寸 | WebP quality | 大小 |
| --- | --- | --- | --- | --- | --- |
| AutoDrive/image-40.png | [one-model-architecture.webp](../../img/2024-07-23-AutoDriveTechChallenge/one-model-architecture.webp) | (12, 170, 1274, 537) | 1100 × 320 | 85 | 24,608 B |
| AutoDrive/image-42.png | [saic-two-model-architecture.webp](../../img/2024-07-23-AutoDriveTechChallenge/saic-two-model-architecture.webp) | (25, 188, 1338, 705) | 1100 × 433 | 85 | 29,870 B |
| CloudControl/image-6.png | [traffic-participant-platforms.webp](../../img/2024-07-21-CloudControlTestGen/traffic-participant-platforms.webp) | (62, 125, 826, 478) | 720 × 333 | 78 | 30,412 B |
| CloudControl/image-14.png | [intersection-conflict-geometry.webp](../../img/2024-07-21-CloudControlTestGen/intersection-conflict-geometry.webp) | (28, 165, 450, 367) | 422 × 202 | 85 | 7,018 B |

两篇文章原先引用的图片总计 14,167,754 B，改为 91,908 B，减少约 99.35%。这是文章引用图片的合计体积，不代表仓库总体积减少；完整原图仍归档在仓库中。
