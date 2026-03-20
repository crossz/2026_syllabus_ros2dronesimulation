**vibe coding**

[忘掉所有武功就练成了](https://www.bilibili.com/video/BV1u44y1W7V3/?spm_id_from=333.337.search-card.all.click&vd_source=b1a0758c4fa58bd173140f614858c591)

**仿真世界**
Gazebo
Rviz2

**传感器感知与数据处理**
机器人如何"看见"周围环境？
- 传感器类型：激光雷达（LiDAR）、深度相机（RGBD）、IMU
- 数据格式：LaserScan、PointCloud2、Image消息类型
- 在Rviz2中配置显示激光雷达扫描数据
- 设置障碍物场景，观察传感器检测效果
- 使用 ros2 bag 录制传感器数据并回放分析

----
## Demo

### turtulebot3



Open Robotics（前身为 OSRF，Open Source Robotics Foundation）是 ROS 的开发和维护组织，也是 TurtleBot 品牌的管理者
 
TurtleBot 3 是 Open Robotics 与 ROBOTIS 联合开发的项目：Open Robotics 负责软件和社区活动，ROBOTIS 负责制造和全球分销
 
TurtleBot 4 是 Open Robotics 与 Clearpath Robotics 深度合作的成果。


仿真+实体机器人
[京东链接](https://search.jd.com/Search?keyword=turtulebot3&enc=utf-8&wq=turtulebot3&pvid=3261e712bd9c43fcb7e6b239431aace1&spmTag=YTAyMTkuYjAwMjM1Ni5jMDAwMDQ2ODkuc2VhcmNoX2NvbmZpcm0)
[B站](https://www.bilibili.com/video/BV1RyzMYpEeS/?spm_id_from=333.337.search-card.all.click&vd_source=b1a0758c4fa58bd173140f614858c591)
#### Gazebo

- 安装turtlebot3模拟器功能包
```bash
sudo apt install ros-${ROS_DISTRO}-turtlebot3*
```

- 安装ros和gazebo桥接工具

```bash
sudo apt install ros-${ROS_DISTRO}-ros-gz
```

- 设置turtlebot3机器人类型环境变量
```bash
export TURTLEBOT3_MODEL=waffle
```

- 启动gazebo仿真环境
```bash
export TURTLEBOT3_MODEL=waffle
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```
- 控制器
```bash
export TURTLEBOT3_MODEL=waffle
ros2 run turtlebot3_teleop teleop_keyboard
```
#### Rivz2
```bash
rviz2
```

- 将`Fixed Frame`选择为`base_footprint`坐标系，放置坐标变换错误
#### Displays窗口“Add” Image显示项
- 选择相机彩色画面话题`/camera/image_raw`
#### Displays窗口“Add” Laserscan
- `LaserScan`的`Topic`话题选择`/sacn`
#### Displays窗口“Add” RobotModel
- 在机器人的`DescriptionTopic`选项中，选择话题`/robot_deseription`

### turtulebot4

[[chapter3-turtulebot4]]


### WPR (商业机器人 demo)

https://deepwiki.com/6-robot/wpr_simulation2

> ROS2 Humble + Gazebo Classic(官方停止维护)

