# TurtleBot3 完整操作指南
## Gazebo Sim + slam_toolbox + Nav2 导航

> **环境：** Ubuntu 22.04 / 24.04 · ROS2 Humble / Jazzy · Gazebo Classic / Harmonic  
> **参考：** [ROBOTIS 官方文档](https://emanual.robotis.com/docs/en/platform/turtlebot3/) · [Nav2 官方文档](https://docs.nav2.org/) · [slam_toolbox GitHub](https://github.com/SteveMacenski/slam_toolbox)

---
==快速上手==：[https://docs.nav2.org/getting_started/index.html](https://docs.nav2.org/getting_started/index.html)

```bash
export TURTLEBOT3_MODEL=waffle_pi
ros2 launch nav2_bringup tb3_simulation_launch.py headless:=False
```

截止到 2026.3.25
有一个bug：
  Summary of the fix:

  The issue was that nav2_minimal_tb3_sim's URDF referenced mesh files at:
  package://nav2_minimal_tb3_sim/models/waffle_base.dae

  But the meshes were actually at:
  package://nav2_minimal_tb3_sim/models/turtlebot3_model/meshes/waffle_base.dae

  Solution: Created symlinks in /opt/ros/jazzy/share/nav2_minimal_tb3_sim/models/ pointing to the correct location.

---

## 📦 前置安装

```bash
# 安装 TurtleBot3 相关包
sudo apt update
sudo apt install -y \
  ros-$ROS_DISTRO-turtlebot3 \
  ros-$ROS_DISTRO-turtlebot3-msgs \
  ros-$ROS_DISTRO-turtlebot3-simulations \
  ros-$ROS_DISTRO-slam-toolbox \
  ros-$ROS_DISTRO-navigation2 \
  ros-$ROS_DISTRO-nav2-bringup \
  ros-$ROS_DISTRO-nav2-map-server

# 添加环境变量到 ~/.bashrc（永久生效）
echo 'export TURTLEBOT3_MODEL=waffle' >> ~/.bashrc
echo 'source /opt/ros/$ROS_DISTRO/setup.bash' >> ~/.bashrc
source ~/.bashrc
```

---

## 第一阶段：启动 Gazebo 仿真环境

### 终端 1 — 启动 Gazebo Sim

```bash
export TURTLEBOT3_MODEL=waffle

# 选择其中一个世界（建图推荐 turtlebot3_world）
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py

# 或者更大的室内环境
# ros2 launch turtlebot3_gazebo turtlebot3_house.launch.py
```

> ✅ 启动后 Gazebo 窗口中可以看到 TurtleBot3 机器人在仿真世界中。

---

## 第二阶段：SLAM 建图（slam_toolbox）

### 配置文件准备

slam_toolbox 使用 **YAML 配置文件**，不使用 Lua 脚本。  
系统默认配置文件位于：

```bash
# /opt/ros/$ROS_DISTRO/share/slam_toolbox/config/mapper_params_online_async.yaml
```

建议复制一份到本地进行自定义：

```bash
mkdir -p ~/tb3_config
cp /opt/ros/$ROS_DISTRO/share/slam_toolbox/config/mapper_params_online_async.yaml \
   ~/tb3_config/slam_toolbox_params.yaml
```

**`~/tb3_config/slam_toolbox_params.yaml` 关键参数说明：**

```yaml
slam_toolbox:
  ros__parameters:

    # 使用仿真时间（Gazebo必须设为true）
    use_sim_time: true

    # SLAM 模式：mapping（建图）/ localization（纯定位）
    mode: mapping

    # 订阅的激光雷达话题
    scan_topic: /scan

    # 坐标系设置
    odom_frame: odom
    map_frame: map
    base_frame: base_footprint

    # 地图分辨率（米/像素）
    resolution: 0.05

    # 最大激光雷达范围（米）
    max_laser_range: 20.0

    # 异步模式：丢弃来不及处理的帧（推荐仿真使用）
    # 同步模式（online_sync）：处理所有帧，精度更高但更慢
    # 此文件对应 online_async 模式

    # 回环检测相关
    minimum_travel_distance: 0.5
    minimum_travel_heading: 0.5
    scan_buffer_size: 10
    scan_buffer_maximum_scan_distance: 10.0
    link_match_minimum_response_fine: 0.1
    distance_decomposition_radius: 3.5
    minimum_time_interval: 0.5

    # 是否启动 RViz（由 launch 文件控制，这里通常不设）
    # do_loop_closing: true
```

### 终端 2 — 启动 slam_toolbox

```bash
# 方式A：使用系统默认配置（快速启动）
#ros2 launch slam_toolbox online_async_launch.py \
#  use_sim_time:=true

# 方式B：使用自定义配置文件（推荐）
ros2 launch slam_toolbox online_async_launch.py \
  use_sim_time:=true \
  slam_params_file:=$HOME/tb3_config/slam_toolbox_params.yaml
```

> ⚠️ **注意：** `online_async_launch.py` 对应异步模式；  
> 如需同步模式（精度更高），改用 `online_sync_launch.py`。

### 终端 3 — 启动 RViz2 可视化
以下二选一：

```bash
# 或使用 TurtleBot3 自带配置
ros2 launch turtlebot3_bringup rviz2.launch.py

# 使用 slam_toolbox 自带的 RViz 配置
# ros2 run rviz2 rviz2 \
#  -d /opt/ros/$ROS_DISTRO/share/slam_toolbox/rviz/mapper_viz.rviz

```

**RViz2 中需要添加的显示项：**

| Display 类型 | Topic | 说明 |
|---|---|---|
| Map | `/map` | 实时建图结果 |
| LaserScan | `/scan` | 激光雷达数据 |
| RobotModel | — | 机器人模型 |
| TF | — | 坐标系变换 |
| Odometry | `/odom` | 里程计轨迹 |

### 终端 4 — 启动键盘遥控（Teleop）

```bash
export TURTLEBOT3_MODEL=waffle
ros2 run turtlebot3_teleop teleop_keyboard
```

**键盘控制：**

```
       w          前进
  a    s    d     左转 / 停止 / 右转
       x          后退

w/x : 增加/减少线速度
a/d : 增加/减少角速度
空格/s : 紧急停止
Ctrl+C : 退出
```

> 🗺️ **建图技巧：**
> - 移动速度要慢（线速度 ≤ 0.2 m/s）
> - 避免急转弯，转弯要缓慢
> - 扫描每个角落，确保地图完整
> - 回到起点触发回环检测，提升地图精度

### 保存地图

当地图建好后，在新终端执行：

```bash
# 创建保存目录
mkdir -p ~/maps

# 保存为 PGM + YAML 格式（供 Nav2 使用）
ros2 run nav2_map_server map_saver_cli -f ~/maps/tb3_map

# 生成文件：
# ~/maps/tb3_map.pgm  — 地图图像
# ~/maps/tb3_map.yaml — 地图元数据
```

**`tb3_map.yaml` 内容示例：**

```yaml
image: tb3_map.pgm
resolution: 0.050000        # 每像素对应的实际距离(米)
origin: [-1.25, -1.25, 0]   # 地图原点坐标
negate: 0
occupied_thresh: 0.65       # 占用阈值
free_thresh: 0.196          # 空闲阈值
```

> 保存完毕后，可以 `Ctrl+C` 关闭 slam_toolbox 和 teleop 节点。  
> **Gazebo 保持运行！**

---

## 第三阶段：Nav2 自主导航

### Nav2 配置文件准备

Nav2 使用统一的 **YAML 参数文件** 配置所有导航组件。  
系统默认配置文件位于：

```bash
#/opt/ros/$ROS_DISTRO/share/turtlebot3_navigation2/param/waffle.yaml
# 或
#/opt/ros/$ROS_DISTRO/share/nav2_bringup/params/nav2_params.yaml
```

复制并自定义：

```bash
cp /opt/ros/$ROS_DISTRO/share/turtlebot3_navigation2/param/waffle.yaml \
   ~/tb3_config/nav2_params.yaml
```

**`~/tb3_config/nav2_params.yaml` 关键参数说明：**

```yaml
amcl:
  ros__parameters:
    use_sim_time: true          # 仿真时间
    alpha1: 0.2                 # 旋转噪声
    alpha2: 0.2
    alpha3: 0.2
    alpha4: 0.2
    alpha5: 0.2
    base_frame_id: "base_footprint"
    beam_skip_distance: 0.5
    global_frame_id: "map"
    laser_model_type: "likelihood_field"
    max_beams: 60
    max_particles: 2000
    min_particles: 500
    odom_frame_id: "odom"
    robot_model_type: "nav2_amcl::DifferentialMotionModel"
    scan_topic: scan

bt_navigator:
  ros__parameters:
    use_sim_time: true
    global_frame: map
    robot_base_frame: base_link
    odom_topic: /odom

controller_server:
  ros__parameters:
    use_sim_time: true
    controller_frequency: 20.0
    # DWB 局部规划器参数
    FollowPath:
      plugin: "dwb_core::DWBLocalPlanner"
      max_vel_x: 0.26
      min_vel_x: -0.26
      max_vel_y: 0.0
      max_vel_theta: 1.0
      min_speed_xy: 0.0
      max_speed_xy: 0.26

planner_server:
  ros__parameters:
    use_sim_time: true
    planner_plugins: ["GridBased"]
    GridBased:
      plugin: "nav2_navfn_planner::NavfnPlanner"
      tolerance: 0.5
      use_astar: false

local_costmap:
  local_costmap:
    ros__parameters:
      use_sim_time: true
      update_frequency: 5.0
      publish_frequency: 2.0
      global_frame: odom
      robot_base_frame: base_link
      rolling_window: true
      width: 3
      height: 3
      resolution: 0.05
      robot_radius: 0.22
      inflation_radius: 0.55

global_costmap:
  global_costmap:
    ros__parameters:
      use_sim_time: true
      update_frequency: 1.0
      publish_frequency: 1.0
      global_frame: map
      robot_base_frame: base_link
      robot_radius: 0.22
      resolution: 0.05
      inflation_radius: 0.55
```

### 终端 2 — 启动 Nav2 导航节点

```bash
export TURTLEBOT3_MODEL=waffle

# 方式A：使用 TurtleBot3 官方 launch（推荐，自动加载 RViz2）
#ros2 launch turtlebot3_navigation2 navigation2.launch.py \
#  use_sim_time:=True \
#  map:=$HOME/maps/tb3_map.yaml

# 方式B：使用自定义参数文件
ros2 launch turtlebot3_navigation2 navigation2.launch.py \
  use_sim_time:=True \
  map:=$HOME/maps/tb3_map.yaml \
  params_file:=$HOME/tb3_config/nav2_params.yaml

# 方式C：使用 nav2_bringup（更底层，灵活）
#ros2 launch nav2_bringup bringup_launch.py \
#  use_sim_time:=True \
#  map:=$HOME/maps/tb3_map.yaml \
#  params_file:=$HOME/tb3_config/nav2_params.yaml
```

> ✅ 启动后会自动打开 **RViz2**，并加载地图。  
> 等待所有节点启动完毕（终端不再滚动输出）后再进行下一步。

### 步骤一：设置初始位姿（必须！）

> AMCL 粒子滤波器需要知道机器人的初始位置才能开始定位。

**在 RViz2 中操作：**

1. 点击工具栏 **`2D Pose Estimate`** 按钮
2. 在地图上**点击机器人实际所在位置**
3. **拖动绿色箭头**指向机器人当前朝向
4. 松开鼠标 → 可以看到地图上出现绿色粒子云

**辅助精确定位（可选但推荐）：**

```bash
# 终端 3 — 临时启动 teleop 辅助定位
ros2 run turtlebot3_teleop teleop_keyboard
```

> 稍微前后移动机器人，观察 RViz2 中绿色粒子云**逐渐收拢**，  
> 说明 AMCL 已经收敛定位成功。  
> 定位完成后 **`Ctrl+C` 关闭 teleop**（避免与 Nav2 冲突）。

### 步骤二：设置导航目标点

#### 方式A：RViz2 手动点击（最直观）

1. 点击 RViz2 工具栏 **`Navigation2 Goal`** 按钮
2. 在地图上**点击目标位置**
3. **拖动绿色箭头**设置到达后的朝向
4. 松开鼠标 → TurtleBot3 **立即开始自动导航** 🚗

> Nav2 自动完成：全局路径规划 → 实时避障 → 到达目标

#### 方式B：命令行发送目标点

```bash
# 发送导航目标（x=1.5, y=0.0, 朝向正前方）
ros2 action send_goal /navigate_to_pose nav2_msgs/action/NavigateToPose \
  "{pose: {header: {frame_id: 'map'}, pose: {position: {x: 1.5, y: 0.0, z: 0.0}, orientation: {w: 1.0}}}}"
```

---

## 第四阶段：进阶演示 — Python 自动巡逻

使用 Nav2 的 `BasicNavigator` Python API 实现多点自动巡逻：

```bash
# 安装 nav2 simple commander
sudo apt install ros-$ROS_DISTRO-nav2-simple-commander
```

**创建巡逻脚本 `~/patrol.py`：**

```python
#!/usr/bin/env python3
"""
TurtleBot3 Nav2 多点巡逻演示
使用 BasicNavigator API（官方推荐方式）
"""
from nav2_simple_commander.robot_navigator import BasicNavigator, TaskResult
from geometry_msgs.msg import PoseStamped
import rclpy
from rclpy.duration import Duration

def make_pose(navigator, x, y, w=1.0):
    """创建 PoseStamped 消息"""
    pose = PoseStamped()
    pose.header.frame_id = 'map'
    pose.header.stamp = navigator.get_clock().now().to_msg()
    pose.pose.position.x = x
    pose.pose.position.y = y
    pose.pose.orientation.w = w
    return pose

def main():
    rclpy.init()
    navigator = BasicNavigator()

    # 等待 Nav2 完全启动
    navigator.waitUntilNav2Active()
    print('✅ Nav2 已就绪！')

    # 设置初始位姿（与 RViz2 中 2D Pose Estimate 等效）
    initial_pose = make_pose(navigator, 0.0, 0.0, 1.0)
    navigator.setInitialPose(initial_pose)

    # 定义巡逻路径点 (x, y, orientation_w)
    waypoints_data = [
        (1.5,  0.0,  1.0),   # 路径点1：正前方
        (1.5,  1.5,  0.707), # 路径点2：右前方
        (0.0,  1.5,  0.0),   # 路径点3：右侧
        (0.0,  0.0,  1.0),   # 路径点4：回到起点
    ]

    waypoints = [make_pose(navigator, x, y, w)
                 for x, y, w in waypoints_data]

    print(f'🚀 开始巡逻，共 {len(waypoints)} 个路径点...')

    # 方式A：依次导航到每个路径点（等待每个完成）
    for i, wp in enumerate(waypoints):
        print(f'📍 前往路径点 {i+1}/{len(waypoints)}: '
              f'({waypoints_data[i][0]}, {waypoints_data[i][1]})')

        navigator.goToPose(wp)

        # 等待导航完成，同时打印进度
        while not navigator.isTaskComplete():
            feedback = navigator.getFeedback()
            if feedback:
                elapsed = Duration.from_msg(
                    feedback.navigation_time).nanoseconds / 1e9
                print(f'  ⏱ 已用时: {elapsed:.1f}s', end='\r')

        result = navigator.getResult()
        if result == TaskResult.SUCCEEDED:
            print(f'\n  ✅ 到达路径点 {i+1}!')
        elif result == TaskResult.CANCELED:
            print(f'\n  ⚠️ 导航被取消')
            break
        elif result == TaskResult.FAILED:
            print(f'\n  ❌ 导航失败，跳过此点')

    # 方式B：一次性发送所有路径点（连续导航）
    # navigator.followWaypoints(waypoints)
    # while not navigator.isTaskComplete():
    #     feedback = navigator.getFeedback()

    print('🎉 巡逻任务完成！')
    navigator.lifecycleShutdown()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

**运行巡逻脚本：**

```bash
python3 ~/patrol.py
```

---

## 📊 各终端分工总览

| 终端 | 阶段 | 命令 |
|------|------|------|
| 终端 1 | 全程 | `ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py` |
| 终端 2 | 建图 | `ros2 launch slam_toolbox online_async_launch.py use_sim_time:=true` |
| 终端 3 | 建图 | `ros2 run rviz2 rviz2 -d /opt/ros/.../mapper_viz.rviz` |
| 终端 4 | 建图 | `ros2 run turtlebot3_teleop teleop_keyboard` |
| 终端 5 | 建图完成 | `ros2 run nav2_map_server map_saver_cli -f ~/maps/tb3_map` |
| 终端 2 | 导航（关闭建图后） | `ros2 launch turtlebot3_navigation2 navigation2.launch.py ...` |
| 终端 3 | 导航辅助 | `ros2 run turtlebot3_teleop teleop_keyboard`（定位后关闭）|
| 终端 4 | 进阶演示 | `python3 ~/patrol.py` |

---

## 🔧 常用调试命令

```bash
# 查看所有活跃 topic
ros2 topic list

# 查看机器人当前定位
ros2 topic echo /amcl_pose --once

# 查看导航状态
ros2 action list

# 查看 TF 树
ros2 run tf2_tools view_frames

# 检查代价地图
ros2 topic echo /global_costmap/costmap_updates

# 取消当前导航
ros2 action send_goal /navigate_to_pose nav2_msgs/action/NavigateToPose "{}" --cancel-after-timeout 1
```

---

## ⚠️ 常见问题排查

| 问题现象 | 可能原因 | 解决方案 |
|---------|---------|---------|
| 地图不更新 | `use_sim_time` 未设置 | 确认所有节点都加了 `use_sim_time:=true` |
| AMCL 粒子不收敛 | 初始位姿设置不准 | 重新点击 `2D Pose Estimate`，稍微移动机器人 |
| 路径规划失败 | 目标点在障碍物内 | 换一个远离墙壁的目标点 |
| 机器人卡在原地 | 代价地图膨胀过大 | 减小 `inflation_radius` 参数 |
| slam_toolbox 报 TF 错误 | `base_frame` 设置错误 | 确认设为 `base_footprint` |
| Nav2 启动后立即崩溃 | 地图文件路径错误 | 检查 `map:=` 参数路径是否正确 |
| `No module named 'catkin_pkg'` | Python 环境冲突 | `sudo apt install python3-catkin-pkg` |

---

## 📚 参考资料

- [ROBOTIS TurtleBot3 SLAM 仿真文档](https://emanual.robotis.com/docs/en/platform/turtlebot3/slam_simulation/)
- [ROBOTIS TurtleBot3 Navigation 仿真文档](https://emanual.robotis.com/docs/en/platform/turtlebot3/nav_simulation/)
- [slam_toolbox GitHub 仓库](https://github.com/SteveMacenski/slam_toolbox)
- [Nav2 官方文档 — SLAM 导航教程](https://docs.nav2.org/tutorials/docs/navigation2_with_slam.html)
- [Nav2 Simple Commander API](https://docs.nav2.org/commander_api/index.html)
