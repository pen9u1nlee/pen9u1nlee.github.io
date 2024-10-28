[ WARN] [1729690633.437380553]: /rtabmap/rtabmap: Did not receive data since 5 seconds! Make sure the input topics are published ("$ rostopic hz my_topic") and the timestamps in their header are set. If topics are coming from different computers, make sure the clocks of the computers are synchronized ("ntpdate"). If topics are not published at the same rate, you could increase "sync_queue_size" and/or "topic_queue_size" parameters (current=10 and 1 respectively).
/rtabmap/rtabmap subscribed to (approx sync):
   /rtabmap/odom \
   /camera/rgb/image_raw \
   /camera/depth/image_raw \
   /camera/rgb/camera_info \
   /rtabmap/odom_info

上述五个话题，两个image_raw的可以正常显示，camera_info可以走rostopic echo显示。odom相关不显示。

定位到节点：rosrun rtabmap_odom rgbd_odometry

注意到节点发布

rosnode info /rtabmap/rgbd_odometry 



--------------------------------------------------------------------------------
Node [/rtabmap/rgbd_odometry]
Publications: 
 * /diagnostics [diagnostic_msgs/DiagnosticArray]
 * /rosout [rosgraph_msgs/Log]
 * /rtabmap/odom [nav_msgs/Odometry]
 * /rtabmap/odom_info [rtabmap_msgs/OdomInfo]
 * /rtabmap/odom_info_lite [rtabmap_msgs/OdomInfo]
 * /rtabmap/odom_last_frame [sensor_msgs/PointCloud2]
 * /rtabmap/odom_local_map [sensor_msgs/PointCloud2]
 * /rtabmap/odom_local_scan_map [sensor_msgs/PointCloud2]
 * /rtabmap/odom_rgbd_image [rtabmap_msgs/RGBDImage]
 * /rtabmap/odom_sensor_data/compressed [rtabmap_msgs/SensorData]
 * /rtabmap/odom_sensor_data/features [rtabmap_msgs/SensorData]
 * /rtabmap/odom_sensor_data/raw [rtabmap_msgs/SensorData]
 * /tf [tf2_msgs/TFMessage]

Subscriptions: 
 * /camera/depth/image_raw [sensor_msgs/Image]
 * /camera/rgb/camera_info [sensor_msgs/CameraInfo]
 * /camera/rgb/image_raw [sensor_msgs/Image]
 * /tf [tf2_msgs/TFMessage]
 * /tf_static [tf2_msgs/TFMessage]

Services: 
 * /rtabmap/pause_odom
 * /rtabmap/reset_odom
 * /rtabmap/reset_odom_to_pose
 * /rtabmap/resume_odom
 * /rtabmap/rgbd_odometry/get_loggers
 * /rtabmap/rgbd_odometry/list
 * /rtabmap/rgbd_odometry/load_nodelet
 * /rtabmap/rgbd_odometry/log_debug
 * /rtabmap/rgbd_odometry/log_error
 * /rtabmap/rgbd_odometry/log_info
 * /rtabmap/rgbd_odometry/log_warning
 * /rtabmap/rgbd_odometry/set_logger_level
 * /rtabmap/rgbd_odometry/unload_nodelet

可复现这个问题：

[ WARN] [1729692277.283890099]: /rgbd_odometry: Did not receive data since 5 seconds! Make sure the input topics are published ("$ rostopic hz my_topic") and the timestamps in their header are set. 
/rgbd_odometry subscribed to (approx sync):
   /rgb/image \
   /depth/image \
   /rgb/camera_info

在看rqt图的时候，不过滤unreachable节点，发现rgbd_odometry是红的

在rtabmap刚启动时，日志报错如下：
[ERROR] [1729692942.694316704]: PluginlibFactory: The plugin for class 'octomap_rviz_plugin/ColorOccupancyGrid' failed to load.  Error: According to the loaded plugin descriptions the class octomap_rviz_plugin/ColorOccupancyGrid with base class type rviz::Display does not exist. Declared types are  rtabmap_rviz_plugins/Info rtabmap_rviz_plugins/MapCloud rtabmap_rviz_plugins/MapGraph rviz/AccelStamped rviz/Axes rviz/Camera rviz/DepthCloud rviz/Effort rviz/FluidPressure rviz/Grid rviz/GridCells rviz/Illuminance rviz/Image rviz/InteractiveMarkers rviz/LaserScan rviz/Map rviz/Marker rviz/MarkerArray rviz/Odometry rviz/Path rviz/PointCloud rviz/PointCloud2 rviz/PointStamped rviz/Polygon rviz/Pose rviz/PoseArray rviz/PoseWithCovariance rviz/Range rviz/RelativeHumidity rviz/RobotModel rviz/TF rviz/Temperature rviz/TwistStamped rviz/WrenchStamped rviz_plugin_tutorials/Imu
[rtabmap/rgbd_odometry-1] process has died [pid 2317056, exit code -11, cmd /opt/ros/noetic/lib/rtabmap_odom/rgbd_odometry --delete_db_on_start rgb/image:=/camera/rgb/image_raw depth/image:=/camera/depth/image_raw rgb/camera_info:=/camera/rgb/camera_info rgbd_image:=rgbd_image_relay odom:=odom imu:=/imu/data __name:=rgbd_odometry __log:=/home/jetson/.ros/log/ce200b02-911d-11ef-af05-7404f1ff6adb/rtabmap-rgbd_odometry-1.log].
log file: /home/jetson/.ros/log/ce200b02-911d-11ef-af05-7404f1ff6adb/rtabmap-rgbd_odometry-1*.log

rgbd_odometry模块启动即挂。

于是按照关键词，检索：`rgbd_odometry die on startup`

https://github.com/introlab/rtabmap_ros/issues/28 用于debug

https://github.com/introlab/rtabmap_ros/issues/628

https://stackoverflow.com/questions/15320267/package-opencv-was-not-found-in-the-pkg-config-search-path

cmake .. -DOPENCV_GENERATE_PKGCONFIG=YES

# ROS和C草的一些问题

卸载ros：https://blog.csdn.net/seniorc/article/details/112276699

将opencv库写进识别文件中：https://blog.csdn.net/jartins/article/details/117027769
sudo vim /etc/ld.so.conf.d/opencv.conf
写入/opt/ros/noetic/lib
sudo ldconfig -v

https://blog.csdn.net/fuhanghang/article/details/130206203

问题主要集中在opencv的适配、rtabmap编译报错、ros启动即挂三个方面。

opencv主要显示的是包的匹配问题。比如装了ABC版本，想让它用A版本却用了B版本，以及找不到适配的头文件。

https://cloud.tencent.com/developer/article/1759523

https://immortalqx.github.io/2021/07/06/opencv-notes-0/

在一台机器上安装多个opencv（不同版本），可以在自己安装时指定不同的目录，然后在~/.bashrc文件里指定不同的参数：
export PKG_CONFIG_PATH=/usr/local/opencv_2.4.9/lib/pkgconfig
export LD_LIBRARY_PATH=/usr/local/opencv_2.4.9/lib

如果想安装到别的地方，可以指定cmake的参数：-D CMAKE_INSTALL_PREFIX=/usr/local/opencv_2.4.9。

在ROS中如何适配不同的opencv版本？

ros启动即挂需要使用ros的debug模式。

# linux动态链接库相关

查看动态链接库的命令：
```sh
ldd <bin-name>
ldd gdbserver

readelf -a <bin-name>|grep library
```

查看动态链接库符号（包括变量名、）的命令：
```sh
nm -n <name> | grep xxx
```



注意事项：动态链接库的调用前提要设置好环境变量？