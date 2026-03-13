
# ==ROS 架构图==[^1]

![[slides/imgs/slide10-ros2-architecture.png]]
```
┌─────────────────────────────────────┐
│         应用程序 (Nodes)              │
├─────────────────────────────────────┤
│    rclcpp (C++) / rclpy (Python)   │
├─────────────────────────────────────┤
│         rcl (C 接口层)               │
├─────────────────────────────────────┤
│      **RMW (中间件接口层)**           │  ← 这一层 interface
├─────────────────────────────────────┤
│    DDS 实现 (Fast DDS, Cyclone DDS) │
└─────────────────────────────────────┘

```

#### ROS2 组成

- **节点(Node)**: 独立运行的程序模块
- **话题(Topic)**: 异步通信机制
- **服务(Service)**: 同步通信机制
- **动作(Action)**: 可取消的同步机制
- **参数(Parameter)**: 节点配置参数

#### 生态系统

- **rcl**: ROS客户端库
- **rclcpp**: C++客户端库，**rclpy**: Python客户端库
- **rviz2**: 3D可视化工具
- **gazebo**: 仿真平台


---


# ROS2 工作空间

## 1. 工作空间概念

- **工作空间(Workspace)**: 存放ROS2项目源码和编译结果的目录
- **src**: 源码目录
- **install**: 安装目录 (编译后)
- **build**: 编译目录 (编译后)
- **log**: 日志目录

```
ros2_ws/
├── src/
│   ├── package1/
│   └── package2/
├── build/
├── install/
└── log/
```

---

## 2. 创建工作空间

### 2.1 新建工作空间

```bash
# 创建目录结构
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws

# 初始化 (可选)
# source /opt/ros/humble/setup.bash
# ros2 pkg create my_package
```

### 2.2 编译工作空间

```bash
# 编译所有包
colcon build

# 编译指定包
colcon build --packages-select my_package

# 编译时安装依赖
colcon build --packages-up-to my_package

# 编译单个包(重新编译)
colcon build --packages-select my_package --cmake-force-configure
```

---

### 2.3 环境设置

```bash
# source install目录
source install/setup.bash

# 自动加载 (添加到.bashrc)
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc

# 检查环境变量
echo $ROS_DISTRO
ros2 pkg list | grep my_package
```

---

## 3. 重叠工作空间

```bash
# 多个工作空间叠加
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash

# 覆盖优先级: 后source的优先
# 移除覆盖
ros2 pkg list | grep -v ~/ros2_ws/install
```

---

# ROS2 功能包

## 1. 功能包概念

- ==**功能包(Package)**: ROS2代码组织的基本单元==
- 每个功能包包含: 源码、配置文件、launch文件、参数文件

### 功能包结构

```
my_package/
├── src/              # C++源码
├── scripts/          # Python脚本
├── include/          # 头文件
├── launch/           # launch文件
├── params/           # 参数文件
├── urdf/             # URDF文件
├── rviz/             # rviz配置
├── CMakeLists.txt   # C++编译配置
└── package.xml       # 包清单
```

---

## 2. 创建功能包

### 2.1 C++功能包

```bash
ros2 pkg create --build-type ament_cmake my_cpp_pkg \
    --dependencies rclcpp std_msgs
```

### 2.2 Python功能包

```bash
ros2 pkg create --build-type ament_python my_py_pkg \
    --dependencies rclpy std_msgs
```

### 2.3 查看功能包信息

```bash
ros2 pkg list                      # 列出所有包
ros2 pkg list | grep <name>       # 查找包
ros2 pkg executables <pkg>         # 列出可执行文件
ros2 pkg prefix <pkg>             # 包安装路径
ros2 pkg xml <pkg>                # 包XML信息
```

---

## 3. 功能包结构示例

### 3.1 C++包结构

```
cpp_package/
├── src/
│   └── my_node.cpp
├── include/
├── CMakeLists.txt
└── package.xml
```

### 3.2 Python包结构

```
py_package/
├── py_package/
│   └── __init__.py
├── resource/
├── test/
├── setup.py
├── setup.cfg
└── package.xml
```

---

# ROS2 节点

## 1. 节点概念

- **节点(Node)**: ROS2中独立运行的可执行程序
- 节点之间通过**话题(Topic)**、**服务(Service)**、**动作(Action)**通信
- 每个节点负责单一功能模块
- 节点可以分布在不同主机上

### 节点特点

- 可执行程序
- 独立运行进程
- 最小化设计原则
- 便于复用和测试

---

## 2. C++ 节点示例

### 2.1 最简单节点

```cpp
#include "rclcpp/rclcpp.hpp"

int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv);
    RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Hello ROS2");
    rclcpp::shutdown();
    return 0;
}
```

### 2.2 完整节点示例

```cpp
#include "rclcpp/rclcpp.hpp"

class MinimalNode : public rclcpp::Node
{
public:
    MinimalNode() : Node("minimal_node")
    {
        RCLCPP_INFO(this->get_logger(), "Node started");
    }
};

int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv);
    rclcpp::spin(std::make_shared<MinimalNode>());
    rclcpp::shutdown();
    return 0;
}
```

---

## 3. Python 节点示例

### 3.1 最简单节点（不推荐）

```python
import rclpy
from rclpy.node import Node

def main(args=None):
    rclpy.init(args=args)
    node = Node('minimal_node')
    node.get_logger().info('Hello ROS2')
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### 3.2 OOP节点（推荐）

```python
import rclpy
from rclpy.node import Node

class MinimalNode(Node):
    def __init__(self):
        super().__init__('minimal_node')
        self.get_logger().info('Node started')

def main(args=None):
    rclpy.init(args=args)
    node = MinimalNode()
    rclpy.spin(node)
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

## 4. 节点操作命令

```bash
# 启动节点
ros2 run <package> <executable>

# 列出运行中的节点
ros2 node list

# 查看节点信息
ros2 node info /<node_name>

# 重映射节点名称
ros2 run pkg node --ros-args -r __node:=new_name

# 查看节点订阅/发布/服务
ros2 node info /<node_name>
```

---

# ROS2 话题通讯

## 1. 话题概念

- **话题(Topic)**: 基于**发布/订阅**的异步通信机制
- 发布者(Publisher)发布消息
- 订阅者(Subscriber)接收消息
- **一对多**、**多对一**通信
- **异步**传输

### 话题特点

- 无需等待响应
- 发布者与订阅者不需要同时运行
- 适合持续数据流
- 实时性要求高的场景

---

## 2. C++ 话题通讯

### 2.1 发布者

```cpp
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"

class PublisherNode : public rclcpp::Node
{
public:
    PublisherNode() : Node("publisher_node"), count_(0)
    {
        publisher_ = this->create_publisher<std_msgs::msg::String>("topic", 10);
        timer_ = this->create_wall_timer(
            500ms, std::bind(&PublisherNode::publish_message, this));
    }

private:
    void publish_message()
    {
        auto message = std_msgs::msg::String();
        message.data = "Hello ROS2 " + std::to_string(count_++);
        RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());
        publisher_->publish(message);
    }

    rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
    rclcpp::TimerBase::SharedPtr timer_;
    size_t count_;
};
```

---

### 2.2 订阅者

```cpp
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"

class SubscriberNode : public rclcpp::Node
{
public:
    SubscriberNode() : Node("subscriber_node")
    {
        subscription_ = this->create_subscription<std_msgs::msg::String>(
            "topic", 10, std::bind(&SubscriberNode::topic_callback, this, std::placeholders::_1));
    }

private:
    void topic_callback(const std_msgs::msg::String::SharedPtr msg)
    {
        RCLCPP_INFO(this->get_logger(), "Received: '%s'", msg->data.c_str());
    }

    rclcpp::Subscription<std_msgs::msg::String>::SharedPtr subscription_;
};
```

---

## 3. Python 话题通讯

### 3.1 发布者

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class PublisherNode(Node):
    def __init__(self):
        super().__init__('publisher_node')
        self.publisher = self.create_publisher(String, 'topic', 10)
        self.timer = self.create_timer(0.5, self.publish_message)
        self.count = 0

    def publish_message(self):
        msg = String()
        msg.data = f'Hello ROS2 {self.count}'
        self.get_logger().info(f'Publishing: "{msg.data}"')
        self.publisher.publish(msg)
        self.count += 1
```

---

### 3.2 订阅者

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class SubscriberNode(Node):
    def __init__(self):
        super().__init__('subscriber_node')
        self.subscription = self.create_subscription(
            String, 'topic', self.topic_callback, 10)

    def topic_callback(self, msg):
        self.get_logger().info(f'Received: "{msg.data}"')
```

---

## 4. 话题命令

```bash
# 列出所有话题
ros2 topic list

# 查看话题详细信息
ros2 topic info /topic_name

# 查看话题消息类型
ros2 topic type /topic_name

# 手动发布消息
ros2 topic pub /topic_name std_msgs/String "data: 'hello'"

# 实时显示话题数据
ros2 topic echo /topic_name

# 查看话题带宽
ros2 topic bw /topic_name

# 查看话题频率
ros2 topic hz /topic_name
```

---

## 5. 话题通讯流程

```
┌─────────────┐         ┌─────────────┐
│  Publisher  │────────▶│   Topic     │
│   Node      │         │             │
└─────────────┘         └──────┬──────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
        ▼                        ▼                        ▼
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│ Subscriber  │         │ Subscriber  │         │ Subscriber  │
│   Node 1    │         │   Node 2    │         │   Node 3    │
└─────────────┘         └─────────────┘         └─────────────┘
```

---


# ROS2 DDS

## 1. DDS 概念

- **DDS** (Data Distribution Service): 数据分发服务
- OMG (Object Management Group) 标准
- ROS2的默认通信中间件

### DDS特点

- 实时性
- 高可靠性
- 发布/订阅机制
- 服务质量(QoS)策略

---

## 2. DDS 实现

### ROS2 支持的DDS

| DDS实现 | 特点 |
|---------|------|
| Fast-DDS | 高性能，开源 |
| CycloneDDS | 高可靠性 |
| Connext | 商业版，高级功能 |

### 切换DDS

```bash
# 使用Fast-DDD (默认)
source /opt/ros/humble/setup.bash

# 使用CycloneDDS
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
source /opt/ros/humble/setup.bash

# 使用Connext
export RMW_IMPLEMENTATION=rmw_connextdds_cpp
```

---

## 3. QoS 策略

### 3.1 常用QoS

| QoS策略 | 说明 |
|---------|------|
| History | 消息历史记录 |
| Depth | 队列深度 |
| Reliability | 可靠性 |
| Durability | 持久性 |
| Deadline | 截止时间 |
| Lifespan | 消息寿命 |

---

### 3.2 QoS示例

```cpp
// C++ QoS配置
rclcpp::QoS qos(10);
qos.reliable();           // 可靠传输
qos.keep_last(10);        // 保留10条
qos.durability_volatile(); // 非持久化

auto publisher = node->create_publisher<std_msgs::msg::String>("topic", qos);
auto subscription = node->create_subscription<std_msgs::msg::String>("topic", qos, callback);
```

```python
# Python QoS配置
from rclpy.qos import QoSProfile, ReliabilityPolicy, HistoryPolicy

qos = QoSProfile(
    reliability=ReliabilityPolicy.RELIABLE,
    history=HistoryPolicy.KEEP_LAST,
    depth=10
)
publisher = node.create_publisher(String, 'topic', qos)
subscription = node.create_subscription(String, 'topic', callback, qos)
```

---

## 4. 查看DDS信息

```bash
# 查看当前DDS实现
ros2 doctor

# 查看RMW实现
ros2 pkg list | grep rmw

# 查看Fast-DDS配置
fastdds
```

---

# ROS2 常用命令工具

## 1. 基础命令

### 1.1 包命令

```bash
# 列出所有包
ros2 pkg list

# 创建包
ros2 pkg create --build-type ament_cmake <pkg_name>
ros2 pkg create --build-type ament_python <pkg_name>

# 查看包信息
ros2 pkg info <pkg_name>

# 列出可执行文件
ros2 pkg executables <pkg_name>
```

### 1.2 节点命令

```bash
# 列出运行中的节点
ros2 node list

# 查看节点信息
ros2 node info /<node_name>

# 运行节点
ros2 run <package> <executable>
```

---

### 1.3 话题命令

```bash
# 列出所有话题
ros2 topic list

# 查看话题信息
ros2 topic info /topic_name

# 发布消息
ros2 topic pub /topic_name <msg_type> "<msg_data>"

# 监听话题
ros2 topic echo /topic_name

# 查看话题带宽/频率
ros2 topic bw /topic_name
ros2 topic hz /topic_name
```

---

### 1.4 服务命令

```bash
# 列出所有服务
ros2 service list

# 查看服务类型
ros2 service type /service_name

# 调用服务
ros2 service call /service_name <srv_type> "<args>"
```

### 1.5 动作命令

```bash
# 列出所有动作
ros2 action list

# 查看动作信息
ros2 action info /action_name

# 发送动作目标
ros2 action send_goal /action_name <action_type> "<goal>" [--feedback]
```

---

## 2. 参数命令

```bash
# 列出参数
ros2 param list

# 获取参数
ros2 param get /node_name param_name

# 设置参数
ros2 param set /node_name param_name value

# 导出参数
ros2 param dump /node_name > file.yaml

# 加载参数
ros2 param load /node_name file.yaml
```

---

## 3. 启动命令

```bash
# 启动launch文件
ros2 launch <package> <launch_file.py>

# 列出可用的launch文件
ros2 launch -s <package>
```

---

## 4. 其他命令

### 4.1 bag命令

```bash
# 录制话题
ros2 bag record /topic1 /topic2

# 播放bag
ros2 bag play <bag_name>

# 查看bag信息
ros2 bag info <bag_name>
```

### 4.2 doctor命令

```bash
# 检查ROS2状态
ros2 doctor

# 检查特定项
ros2 doctor -r <component>
```

---

# ROS2 Rviz2使用

## 1. Rviz2 简介

- **Rviz2**: ROS2的3D可视化工具
- 可视化传感器数据
- 显示机器人模型
- 调试导航、定位等功能

### 启动Rviz2

```bash
# 启动
rviz2

# 加载配置文件
rviz2 -d <config_file.rviz>
```

---

## 2. Rviz2 界面

### 2.1 界面区域

- **左侧**: 显示项列表(Displays)
- **中间**: 3D视图区
- **右侧**: 属性面板/工具栏
- **底部**: 时间面板

### 2.2 常用显示类型

| 显示类型 | 用途 |
|----------|------|
| RobotModel | 显示机器人模型 |
| Map | 显示地图 |
| LaserScan | 激光雷达数据 |
| PointCloud | 点云数据 |
| Image | 相机图像 |
| Path | 路径显示 |
| Pose | 位姿显示 |

---

## 3. 配置示例

### 3.1 添加RobotModel

1. 点击"Add"按钮
2. 选择"RobotModel"
3. 设置"Robot Description"参数
4. 输入URDF内容或URL

### 3.2 添加LaserScan

1. Add → LaserScan
2. 设置"Topic"
3. 调整颜色/样式

### 3.3 保存配置

1. File → Save Config As
2. 保存.rviz文件
3. 下次启动: `rviz2 -d config.rviz`

---

## 4. 常用快捷键

| 快捷键 | 功能 |
|--------|------|
| 鼠标左键 | 旋转视角 |
| 鼠标中键 | 平移视角 |
| 鼠标右键 | 缩放 |
| t | 顶部视图 |
| p | 侧面视图 |
| f | 聚焦选中物体 |

---

# ROS2 Rqt工具箱

## 1. Rqt 简介

- **Rqt**: ROS2的GUI开发工具
- 基于Qt的插件系统
- 可动态加载各种工具

### 启动Rqt

```bash
# 启动rqt
rqt

# 启动特定插件
rqt --plugin rqt_topic
```

---

## 2. 常用插件

### 2.1 Topic Monitor

- 监控话题消息
- 显示实时数据
- 录制消息

```bash
rqt --plugin rqt_topic
```

### 2.2 Service Caller

- 调用服务
- 调试服务

```bash
rqt --plugin rqt_service_caller
```

---

### 2.3 Node Graph

- 显示节点关系图
- 查看通信拓扑

```bash
rqt --plugin rqt_graph
```

### 2.4 Parameter Reconfigure

- 动态调整参数

```bash
rqt --plugin rqt_reconfigure
```

---

## 3. 安装更多插件

```bash
# 安装完整rqt工具集
sudo apt install ros-humble-rqt-*

# 常用插件
sudo apt install ros-humble-rqt-graph ros-humble-rqt-plot
sudo apt install ros-humble-rqt-image-view ros-humble-rqt-bag
```

---

# ROS2 Launch启动文件配置

## 1. Launch文件概念

- **Launch文件**: 用于同时启动多个节点
- XML/YAML/Python格式
- 自动配置环境

### Launch文件位置

```
package/
├── launch/
│   └── my_launch.py
│   └── my_launch.xml
│   └── my_launch.yaml
```

---

## 2. Python Launch文件

### 2.1 基本结构

```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='demo_nodes_cpp',
            executable='talker',
            name='my_talker'
        ),
        Node(
            package='demo_nodes_cpp',
            executable='listener',
            name='my_listener'
        )
    ])
```

---

### 2.2 参数传递

```python
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration
from launch_ros.actions import Node

def generate_launch_description():
    declare_arg = DeclareLaunchArgument(
        'param_name',
        default_value='default',
        description='参数说明'
    )

    param = LaunchConfiguration('param_name')

    node = Node(
        package='my_package',
        executable='my_node',
        parameters=[{'param_name': param}]
    )

    return LaunchDescription([declare_arg, node])
```

---

### 2.3 包含其他Launch

```python
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource

def generate_launch_description():
    include1 = IncludeLaunchDescription(
        PythonLaunchDescriptionSource([
            '/path/to/launch1.launch.py'
        ])
    )

    include2 = IncludeLaunchDescription(
        PythonLaunchDescriptionSource([
            '/path/to/launch2.launch.py'
        ]),
        launch_arguments={'arg1': 'value'}.items()
    )

    return LaunchDescription([include1, include2])
```

---

## 3. XML Launch文件

```xml
<launch>
  <arg name="param_name" default="default" doc="参数说明"/>

  <node pkg="demo_nodes_cpp" exec="talker" name="my_talker">
    <param name="param_name" value="$(var param_name)"/>
  </node>

  <node pkg="demo_nodes_cpp" exec="listener" name="my_listener"/>

  <include file="$(find-pkg-share pkg)/launch/other.launch.xml"/>
</launch>
```

---

## 4. Launch命令

```bash
# 启动launch文件
ros2 launch <package> <launch_file>

# 列出可用launch文件
ros2 launch -s <package>

# 带参数启动
ros2 launch pkg launch.py arg1:=value1 arg2:=value2
```

---

# ROS2 录制回放工具

## 1. Rosbag2 简介

- **Rosbag2**: ROS2的话题录制工具
- 录制话题数据
- 回放录制的数据
- 调试和测试

---

## 2. 录制命令

### 2.1 基本录制

```bash
# 录制所有话题
ros2 bag record -a

# 录制指定话题
ros2 bag record /topic1 /topic2

# 指定输出文件名
ros2 bag record -o my_bagfile /topic1

# 指定输出目录
ros2 bag record -s sqlite3 -o my_bagfile /topic1
```

### 2.2 录制选项

```bash
# 录制有限数量的消息
ros2 bag record -a --max-split-size 100MB

# 压缩录制
ros2 bag record -a --compression-mode file
ros2 bag record -a --compression-format zstd
```

---

## 3. 回放命令

### 3.1 基本回放

```bash
# 查看bag信息
ros2 bag info <bag_name>

# 回放
ros2 bag play <bag_name>

# 循环回放
ros2 bag play -l <bag_name>
```

### 3.2 回放选项

```bash
# 指定起始点
ros2 bag play --start-offset 1000 <bag_name>

# 暂停后自动播放
ros2 bag play -a <bag_name>

# 改变发布速率
ros2 bag play -r 2.0 <bag_name>  # 2倍速

# 回放指定话题
ros2 bag play --topics /topic1 /topic2 <bag_name>
```

---

## 4. 常用命令汇总

```bash
# 录制
ros2 bag record <options> <topics>

# 回放
ros2 bag play <options> <bag>

# 信息
ros2 bag info <bag>

# 压缩
ros2 bag compress <bag>

# 修复
ros2 bag fix <bag>
```

---

# ROS2 URDF模型

## 1. URDF 简介

- **URDF** (Unified Robot Description Format): 机器人描述格式
- XML格式
- 描述机器人连杆和关节

### URDF文件位置

```
package/
├── urdf/
│   └── robot.urdf
├── rviz/
│   └── robot.rviz
└── launch/
    └── robot.launch.py
```

---

## 2. URDF基本结构

### 2.1 完整示例

```xml
<?xml version="1.0"?>
<robot name="my_robot">

  <!-- 连杆定义 -->
  <link name="base_link">
    <visual>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <box size="0.5 0.2 0.1"/>
      </geometry>
      <material name="blue">
        <color rgba="0 0 0.8 1"/>
      </material>
    </visual>
    <collision>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <box size="0.5 0.2 0.1"/>
      </geometry>
    </collision>
  </link>

  <!-- 关节定义 -->
  <joint name="joint1" type="revolute">
    <parent link="base_link"/>
    <child link="link1"/>
    <origin xyz="0.25 0 0.1" rpy="0 0 0"/>
    <axis xyz="0 0 1"/>
    <limit lower="-1.57" upper="1.57" effort="10" velocity="5"/>
  </joint>

</robot>
```

---

## 3. 机器人状态发布器

### 3.1 C++ 状态发布器

```cpp
#include "robot_state_publisher/robot_state_publisher.h"

// 使用
#include <kdl_parser/kdl_parser.hpp>
#include <urdf/model.h>

// 加载URDF并发布状态
robot_state_publisher::RobotStatePublisher rsp(node);
rsp.publishTransforms(jnt_state, now);
```

### 3.2 Python 状态发布器

```python
from robot_state_publisher.robot_state_publisher import RobotStatePublisher

# 创建发布器
rsp = RobotStatePublisher(node, kdl_tree, None)

# 发布
rsp.publish_transforms(jnt_state)
```

---

## 4. 启动URDF

```python
# launch文件
from launch import LaunchDescription
from launch_ros.actions import Node
from launch.substitutions import Command
from launch_ros.parameter_descriptions import ParameterValue

def generate_launch_description():
    robot_description = Command([
        'xacro ',
        '/path/to/robot.urdf.xacro'
    ])

    robot_state_publisher = Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        parameters=[{'robot_description': robot_description}]
    )

    return LaunchDescription([robot_state_publisher])
```

---

## 5. 验证URDF

```bash
# 检查URDF语法
check_urdf robot.urdf

# 可视化URDF
urdf_to_graphviz robot.urdf
```

---

# ROS2 Gazebo仿真平台

## 1. Gazebo 简介

- **Gazebo**: 机器人仿真平台
- 物理引擎支持
- 传感器仿真
- 环境仿真

### 安装

```bash
# 安装Gazebo
sudo apt install ros-humble-gazebo-*

# 启动Gazebo
gazebo
```

---

## 2. 创建仿真环境

### 2.1 世界文件

```xml
<?xml version="1.0" ?>
<sdf version="1.6">
  <world name="default">
    <!-- 物理插件 -->
    <physics type="ode">
      <max_step_size>0.001</max_step_size>
      <real_time_factor>1</real_time_factor>
    </physics>

    <!-- 地面 -->
    <model name="ground_plane">
      <static>true</static>
      <link name="link">
        <collision>
          <geometry>
            <plane>
              <normal>0 0 1</normal>
            </plane>
          </geometry>
        </collision>
      </link>
    </model>

  </world>
</sdf>
```

---

## 3. ROS2与Gazebo集成

### 3.1 安装桥接包

```bash
sudo apt install ros-humble-gazebo-ros-pkgs
```

### 3.2 启动仿真

```bash
# 启动Gazebo
ros2 launch gazebo_ros launch/gazebo.launch.py

# 带机器人启动
ros2 launch gazebo_ros launch/robot.launch.py
```

---

## 4. 常用命令

```bash
# 创建空世界
gazebo -s libgazebo_ros_init.so

# 使用指定世界文件
gazebo -s libgazebo_ros_init.so worlds/empty.world

# 使用ros_gz桥接
ros2 run ros_gz bridge <bridge_config>
```

---

# ROS2 TF2坐标变换

## 1. TF2 简介

- **TF2**: 机器人坐标变换库
- 管理多坐标系关系
- 跟踪位置随时间变化

### TF2特点

- 分布式
- 实时性好
- 自动插值

---

## 2. C++ TF2 使用

### 2.1 广播变换

```cpp
#include "tf2_ros/transform_broadcaster.h"

class TransformBroadcasterNode : public rclcpp::Node
{
public:
    TransformBroadcasterNode() : Node("tf_broadcaster")
    {
        tf_broadcaster_ = std::make_unique<tf2_ros::TransformBroadcaster>(*this);
    }

    void broadcast_transform()
    {
        geometry_msgs::msg::TransformStamped t;

        t.header.stamp = this->get_clock()->now();
        t.header.frame_id = "world";
        t.child_frame_id = "turtle1";

        t.transform.translation.x = 1.0;
        t.transform.translation.y = 0.0;
        t.transform.translation.z = 0.0;

        t.transform.rotation.x = 0.0;
        t.transform.rotation.y = 0.0;
        t.transform.rotation.z = 0.0;
        t.transform.rotation.w = 1.0;

        tf_broadcaster_->sendTransform(t);
    }

private:
    std::unique_ptr<tf2_ros::TransformBroadcaster> tf_broadcaster_;
};
```

---

### 2.2 监听变换

```cpp
#include "tf2_ros/transform_listener.h"
#include "tf2_ros/buffer.h"

class TransformListenerNode : public rclcpp::Node
{
public:
    TransformListenerNode() : Node("tf_listener")
    {
        tf_buffer_ = std::make_shared<tf2_ros::Buffer>(this->get_clock());
        tf_listener_ = std::make_shared<tf2_ros::TransformListener>(*tf_buffer_, this);
    }

    void lookup_transform()
    {
        try {
            auto t = tf_buffer_->lookupTransform(
                "target_frame",
                "source_frame",
                tf2::TimePointZero);
            RCLCPP_INFO(this->get_logger(), "Transform: (%.2f, %.2f, %.2f)",
                t.transform.translation.x,
                t.transform.translation.y,
                t.transform.translation.z);
        } catch (const tf2::TransformException &ex) {
            RCLCPP_WARN(this->get_logger(), "%s", ex.what());
        }
    }

private:
    std::shared_ptr<tf2_ros::Buffer> tf_buffer_;
    std::shared_ptr<tf2_ros::TransformListener> tf_listener_;
};
```

---

## 3. Python TF2 使用

### 3.1 广播变换

```python
import rclpy
from rclpy.node import Node
from tf2_ros import TransformBroadcaster
from geometry_msgs.msg import TransformStamped

class TFBroadcaster(Node):
    def __init__(self):
        super().__init__('tf_broadcaster')
        self.tf_broadcaster = TransformBroadcaster(self)

    def broadcast_transform(self):
        t = TransformStamped()
        t.header.stamp = self.get_clock().now().to_msg()
        t.header.frame_id = 'world'
        t.child_frame_id = 'turtle1'
        t.transform.translation.x = 1.0
        t.transform.translation.y = 0.0
        t.transform.translation.z = 0.0
        t.transform.rotation.x = 0.0
        t.transform.rotation.y = 0.0
        t.transform.rotation.z = 0.0
        t.transform.rotation.w = 1.0
        self.tf_broadcaster.sendTransform(t)
```

---

### 3.2 监听变换

```python
import rclpy
from rclpy.node import Node
from tf2_ros import TransformListener, Buffer
from geometry_msgs.msg import TransformStamped

class TFListener(Node):
    def __init__(self):
        super().__init__('tf_listener')
        self.tf_buffer = Buffer()
        self.tf_listener = TransformListener(self.tf_buffer, self)

    def lookup_transform(self):
        try:
            t = self.tf_buffer.lookup_transform(
                'target_frame',
                'source_frame',
                rclpy.time.Time())
            self.get_logger().info(f'Transform: {t.transform.translation}')
        except Exception as e:
            self.get_logger().warn(f'{e}')
```

---

## 4. TF2 命令行工具

```bash
# 列出所有坐标系
ros2 run tf2_ros view_frames

# 监听变换
ros2 run tf2_ros tf2_echo source_frame target_frame

# 静态变换
ros2 run tf2_ros static_transform_publisher x y z roll pitch yaw parent child
```

---

## 5. TF2 静态变换

### 5.1 静态广播器

```cpp
#include "tf2_ros/static_transform_broadcaster.h"

auto static_broadcaster = std::make_shared<tf2_ros::StaticTransformBroadcaster>(node);
geometry_msgs::msg::TransformStamped t;
// ... 设置变换
static_broadcaster->sendTransform(t);
```

### 5.2 启动文件中的静态变换

```python
from launch_ros.actions import Node

Node(
    package='tf2_ros',
    executable='static_transform_publisher',
    arguments=['1', '0', '0', '0', '0', '0', 'world', 'base_link']
)
```

[^1]: RMW 还有一层抽象
