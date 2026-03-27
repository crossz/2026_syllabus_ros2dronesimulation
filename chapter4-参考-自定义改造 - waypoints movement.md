

## 🛤️ 第四步：创建航线跟随节点（核心代码）

### 4.1 创建 ROS2 功能包

```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python waypoint_mission \
  --dependencies rclpy nav2_simple_commander geometry_msgs
```

### 4.2 编写航线跟随节点

````python
# waypoint_mission/waypoint_mission/mission_node.py

import rclpy
from rclpy.node import Node
from nav2_simple_commander.robot_navigator import BasicNavigator, TaskResult
from geometry_msgs.msg import PoseStamped
from rclpy.duration import Duration
import time

def create_pose(navigator, x, y, yaw_z=0.0, yaw_w=1.0):
    """创建 PoseStamped 航点"""
    pose = PoseStamped()
    pose.header.frame_id = 'map'
    pose.header.stamp = navigator.get_clock().now().to_msg()
    pose.pose.position.x = x
    pose.pose.position.y = y
    pose.pose.position.z = 0.0
    pose.pose.orientation.z = yaw_z
    pose.pose.orientation.w = yaw_w
    return pose

def main():
    rclpy.init()
    navigator = BasicNavigator()

    # ✅ Step 1: 设置初始位置
    initial_pose = create_pose(navigator, 0.0, 0.0, 0.0, 1.0)
    navigator.setInitialPose(initial_pose)

    # ✅ Step 2: 等待 Nav2 完全启动
    navigator.waitUntilNav2Active()

    # ✅ Step 3: 定义航线（waypoints）
    waypoints = [
        create_pose(navigator,  1.0,  0.0, 0.0, 1.0),   # 航点 1
        create_pose(navigator,  1.0,  1.0, 0.707, 0.707), # 航点 2（转向90°）
        create_pose(navigator,  0.0,  1.0, 1.0, 0.0),    # 航点 3（转向180°）
        create_pose(navigator,  0.0,  0.0, 0.0, 1.0),    # 返回起点
    ]

    # ✅ Step 4: 执行航线跟随
    navigator.followWaypoints(waypoints)

    # ✅ Step 5: 监控执行状态
    i = 0
    while not navigator.isTaskComplete():
        feedback = navigator.getFeedback()
        if feedback and i % 5 == 0:
            print(f'[航点跟随] 当前航点: {feedback.current_waypoint + 1}/{len(waypoints)}')
        i += 1
        time.sleep(0.5)

    # ✅ Step 6: 检查结果
    result = navigator.getResult()
    if result == TaskResult.SUCCEEDED:
        print('✅ 航线执行完成！')
    elif result == TaskResult.CANCELED:
        print('⚠️ 任务被取消')
    elif result == TaskResult.FAILED:
        print('❌ 任务执行失败')

    rclpy.shutdown()

if __name__ == '__main__':
    main()
````

### 4.3 配置 setup.py

````python
# setup.py
from setuptools import setup

package_name = 'waypoint_mission'

setup(
    name=package_name,
    version='0.0.1',
    packages=[package_name],
    install_requires=['setuptools'],
    entry_points={
        'console_scripts': [
            'mission_node = waypoint_mission.mission_node:main',
        ],
    },
)
````

---

## 🔧 第五步：编译与运行

```bash
# 编译
cd ~/ros2_ws
colcon build --packages-select waypoint_mission
source install/setup.bash

# 运行（确保 Nav2 已启动）
ros2 run waypoint_mission mission_node
```

---

## 🚀 第六步：进阶扩展

### 6.1 从 YAML 文件加载航线

````python
# waypoints_config.yaml
waypoints:
  - {x: 1.0, y: 0.0, yaw: 0.0}
  - {x: 1.0, y: 1.0, yaw: 1.57}
  - {x: 0.0, y: 1.0, yaw: 3.14}
  - {x: 0.0, y: 0.0, yaw: 0.0}
````

````python
import yaml
from math import sin, cos

def load_waypoints_from_yaml(navigator, file_path):
    with open(file_path, 'r') as f:
        config = yaml.safe_load(f)
    
    waypoints = []
    for wp in config['waypoints']:
        yaw = wp['yaw']
        pose = create_pose(
            navigator, 
            wp['x'], wp['y'],
            sin(yaw / 2),  # quaternion z
            cos(yaw / 2)   # quaternion w
        )
        waypoints.append(pose)
    return waypoints
````

### 6.2 添加航点任务插件（到达每个航点后执行动作）

```bash
# Nav2 支持 WaypointTaskExecutor 插件
# 可在到达航点后执行：拍照、等待、发布消息等
```

````yaml
# nav2_params.yaml 中配置
waypoint_follower:
  ros__parameters:
    loop_rate: 20
    stop_on_failure: false
    waypoint_task_executor_plugin: "wait_at_waypoint"
    wait_at_waypoint:
      plugin: "nav2_waypoint_follower::WaitAtWaypoint"
      enabled: True
      waypoint_pause_duration: 2000  # 毫秒
````

---

## 📊 完整开发流程总结

| 步骤 | 内容 | 工具/包 |
|------|------|---------|
| 1️⃣ | 环境安装 | `turtlebot3`, `nav2`, `slam-toolbox` |
| 2️⃣ | SLAM 建图 | `cartographer` + `teleop_keyboard` |
| 3️⃣ | 保存地图 | `map_saver_cli` |
| 4️⃣ | 启动导航栈 | `navigation2.launch.py` |
| 5️⃣ | 编写航线节点 | `nav2_simple_commander` |
| 6️⃣ | 加载航线配置 | YAML 文件 |
| 7️⃣ | 扩展任务插件 | `WaypointTaskExecutor` |

---

## 📚 参考资源

| 资源 | 链接 |
|------|------|
| 📖 Nav2 官方文档 | [docs.nav2.org](https://docs.nav2.org/tutorials/docs/navigation2_on_real_turtlebot3.html) |
| 📖 ROBOTIS TurtleBot3 文档 | [emanual.robotis.com](https://emanual.robotis.com/docs/en/platform/turtlebot3/navigation/) |
| 💻 Nav2 Waypoint Follower 示例代码 | [GitHub - example_waypoint_follower.py](https://github.com/ros-planning/navigation2/blob/main/nav2_simple_commander/nav2_simple_commander/example_waypoint_follower.py) |
| 🎥 Nav2 Waypoint Follower 视频教程 | [YouTube - ROS World 2020](https://www.youtube.com/watch?v=F2h7ZuJW8y0) |
| 📖 Waypoint Follower 配置文档 | [docs.nav2.org/configuration](https://docs.nav2.org/configuration/packages/configuring-waypoint-follower.html) |
| 🎥 Nav2 完整入门课程 | [YouTube - 1 Hour Crash Course](https://www.youtube.com/watch?v=idQb2pB-h2Q) |

---

> 💡 **提示**：如果你需要在 **Gazebo 仿真** 中测试，建议先用 `turtlebot3_world` 场景熟悉流程，再迁移到真实机器人。真实机器人需要额外配置 LiDAR、里程计等传感器标定。