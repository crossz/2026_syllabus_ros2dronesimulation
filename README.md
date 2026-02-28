---
marp: true
theme: default
paginate: true
header: ROS2+PX4 无人机仿真实践
---

# ROS2 简介

## ROS2 机器人操作系统

### 1. ROS2 概念

- **ROS** (Robot Operating System) 是用于机器人软件开发的**开源中间件**
- **ROS2** 是 ROS 的新一代版本，于2020年正式发布
- ROS2 并非 ROS 的小幅度更新，而是**完全重写**

---

### 2. ROS 发展历程

| 版本 | 发布时间 | 特性 |
| ------ | ---------- | ------ |
| ROS1 | 2010年 | 科研为主，实时性差 |
| ROS2 | 2020年 | 工业级，实时性强 |
| ROS2 Humble | 2022年 | 长期支持版本 LTS |

---

### 3. ROS2  vs  ROS1

#### ROS1 缺点

- 通信基于TCPROS/UDPROS，实时性差
- 无实时操作系统(RTOS)支持
- 核心API不稳定
- 几乎只支持Linux，集成开发工具不完善

#### ROS2 改进

- 基于DDS(数据分发服务)通信
- 支持RTOS (Real-Time OS)
- 核心API稳定
- 支持多平台 (Linux/Windows/macOS)
- 集成工具完善，安全性设计

---

### 4. ROS2 组成

#### 4.1 计算图

- **节点(Node)**: 独立运行的程序模块
- **话题(Topic)**: 异步通信机制
- **服务(Service)**: 同步通信机制
- **动作(Action)**: 可取消的同步机制
- **参数(Parameter)**: 节点配置参数

#### 4.2 生态系统

- **rcl**: ROS客户端库
- **rclcpp**: C++客户端库，**rclpy**: Python客户端库
- **rviz2**: 3D可视化工具
- **gazebo**: 仿真平台

---

# ROS2 安装 (Humble)

## 1. 系统要求

- **Ubuntu**: 22.04 (推荐)
- **内存**: 最小 2GB，建议 4GB+
- **磁盘空间**: 至少 50GB

## 2. 安装步骤

### 2.1 设置语言环境

```bash
locale  # 检查语言环境
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

---

### 2.2 添加ROS2源

```bash
# 安装依赖工具
sudo apt install software-properties-common
sudo add-apt-repository universe

# 添加ROS2 GPG密钥
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

# 添加源
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

---

### 2.3 安装ROS2包

```bash
# 更新apt
sudo apt update

# 桌面版安装 (推荐)
sudo apt install ros-humble-desktop

# 基础版安装
sudo apt install ros-humble-ros-base

# 开发工具
sudo apt install ros-dev-tools
```

### 2.4 环境设置

```bash
# 自动source
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

---

### 2.5 验证安装

```bash
# 检查ROS2环境变量
printenv | grep ROS

# 启动小海龟demo
ros2 run turtlesim turtle_simulator

# 启动键盘控制(新终端)
ros2 run turtlesim turtle_teleop_key
```

---

# ROS2 集成开发环境搭建

## 1. VSCode 安装与配置

### 1.1 安装VSCode

### 1.2 常用插件

| 插件 | 功能 |
|------|------|
| C/C++ | C/C++代码补全、调试 |
| Python | Python开发支持 |
| ROS | ROS功能支持 |
| Markdown All in One | Markdown编辑 |
| Marp | PPT预览 |

---

### 1.3 VSCode ROS配置

```json
{
    "ROS.Distro": "humble",
    "ros.distro": "humble",
    "C_Cpp.default.configurationProvider": "ms-vscode.cpptools"
}
```

### 1.4 创建ROS2工作空间

```bash
# 创建工作空间
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
colcon build
source install/setup.bash
```

---

## 2. C++ 开发环境（简介）

所有开发的基础环境

- 操作系统: Ubuntu 22.04 LTS (Jammy Jellyfish)。
- 编译与构建工具:
build-essential: 包含GCC, G++, make等基础编译工具。
cmake: 跨平台的构建系统生成工具，ROS 2项目必备。
git: 版本控制工具，用于克隆代码仓库。
- 其他工具:
python3-colcon-common-extensions: Colcon构建工具的扩展，更便捷的命令。
python3-rosdep: ROS依赖管理工具。

---

## 3. Git配置

```bash
# 配置Git
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# SSH密钥
ssh-keygen -t ed25519 -C "your@email.com"
cat ~/.ssh/id_ed25519.pub  # 添加到GitHub
```

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

- **功能包(Package)**: ROS2代码组织的基本单元
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

# ROS2 服务通讯

## 1. 服务概念

- **服务(Service)**: 基于**请求/响应**的同步通信机制
- 客户端发送请求
- 服务器处理后返回响应
- **一对一**通信
- **同步**传输

### 服务特点

- 双向通信
- 有返回值
- 适合一次性操作
- 不适合持续数据流

---

## 2. 自定义服务类型

### 2.1 创建服务接口

```bash
# 创建接口功能包
ros2 pkg create --build-type ament_cmake example_interfaces

# 创建服务接口文件
mkdir -p example_interfaces/srv
```

### 2.2 服务接口定义

```yaml
# example_interfaces/srv/AddTwoInts.srv
int
int64 b64 a
---
int64 sum
```

### 2.3 CMakeLists.txt配置

```cmake
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "srv/AddTwoInts.srv"
)
```

### 2.4 package.xml配置

```xml
<build_depend>rosidl_default_generators</build_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```

---

## 3. C++ 服务通讯

### 3.1 服务端

```cpp
#include "rclcpp/rclcpp.hpp"
#include "example_interfaces/srv/add_two_ints.hpp"

class AddTwoIntsServer : public rclcpp::Node
{
public:
    AddTwoIntsServer() : Node("add_two_ints_server")
    {
        service_ = this->create_service<example_interfaces::srv::AddTwoInts>(
            "add_two_ints",
            std::bind(&AddTwoIntsServer::handle_service, this, std::placeholders::_1, std::placeholders::_2));
    }

private:
    void handle_service(
        const std::shared_ptr<example_interfaces::srv::AddTwoInts::Request> request,
        std::shared_ptr<example_interfaces::srv::AddTwoInts::Response> response)
    {
        response->sum = request->a + request->b;
        RCLCPP_INFO(this->get_logger(), "%ld + %ld = %ld",
            request->a, request->b, response->sum);
    }

    rclcpp::Service<example_interfaces::srv::AddTwoInts>::SharedPtr service_;
};
```

---

### 3.2 客户端

```cpp
#include "rclcpp/rclcpp.hpp"
#include "example_interfaces/srv/add_two_ints.hpp"

class AddTwoIntsClient : public rclcpp::Node
{
public:
    AddTwoIntsClient() : Node("add_two_ints_client")
    {
        client_ = this->create_client<example_interfaces::srv::AddTwoInts>("add_two_ints");
    }

    void send_request(int64_t a, int64_t b)
    {
        auto request = std::make_shared<example_interfaces::srv::AddTwoInts::Request>();
        request->a = a;
        request->b = b;

        while (!client_->wait_for_service(1.0s)) {
            if (!rclcpp::ok()) {
                RCLCPP_ERROR(this->get_logger(), "Interrupted while waiting");
                return;
            }
            RCLCPP_INFO(this->get_logger(), "Service not available");
        }

        auto result = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(result) == rclcpp::FutureReturnCode::SUCCESS) {
            RCLCPP_INFO(this->get_logger(), "Result: %ld", result.get()->sum);
        } else {
            RCLCPP_ERROR(this->get_logger(), "Failed to call service");
        }
    }

private:
    rclcpp::Client<example_interfaces::srv::AddTwoInts>::SharedPtr client_;
};
```

---

## 4. Python 服务通讯

### 4.1 服务端

```python
import rclpy
from rclpy.node import Node
from example_interfaces.srv import AddTwoInts

class AddTwoIntsServer(Node):
    def __init__(self):
        super().__init__('add_two_ints_server')
        self.srv = self.create_service(AddTwoInts, 'add_two_ints', self.add_two_ints_callback)

    def add_two_ints_callback(self, request, response):
        response.sum = request.a + request.b
        self.get_logger().info(f'{request.a} + {request.b} = {response.sum}')
        return response
```

---

### 4.2 客户端

```python
import rclpy
from rclpy.node import Node
from example_interfaces.srv import AddTwoInts

class AddTwoIntsClient(Node):
    def __init__(self):
        super().__init__('add_two_ints_client')
        self.client = self.create_client(AddTwoInts, 'add_two_ints')

    def send_request(self, a, b):
        while not self.client.wait_for_service(1.0):
            self.get_logger().info('Service not available')
        request = AddTwoInts.Request()
        request.a = a
        request.b = b
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        return future.result().sum
```

---

## 5. 服务命令

```bash
# 列出所有服务
ros2 service list

# 查看服务类型
ros2 service type /service_name

# 手动调用服务
ros2 service call /service_name example_interfaces/srv/AddTwoInts "{a: 5, b: 3}"

# 查找服务提供者
ros2 service find <type>
```

---

# ROS2 动作通讯

## 1. 动作概念

- **动作(Action)**: 可取消的异步通信机制
- 包含: **目标(Goal)**、**反馈(Feedback)**、**结果(Result)**
- 适合长时间运行任务
- 可以取消执行

### 动作特点

- 可取消
- 有进度反馈
- 长时间运行任务
- 有最终结果

---

## 2. 自定义动作类型

### 2.1 创建动作接口

```yaml
# example_interfaces/action/Fibonacci.action
int32 order
---
int32[] sequence
---
int32[] partial_sequence
```

### 2.2 CMakeLists.txt配置

```cmake
rosidl_generate_interfaces(${PROJECT_NAME}
  "action/Fibonacci.action"
  DEPENDENCIES std_msgs
)
```

### 2.3 package.xml配置

```xml
<build_depend>rosidl_default_generators</build_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```

---

## 3. C++ 动作通讯

### 3.1 动作服务端

```cpp
#include "rclcpp/rclcpp.hpp"
#include "rclcpp_action/rclcpp_action.hpp"
#include "example_interfaces/action/fibonacci.hpp"

class FibonacciActionServer : public rclcpp::Node
{
public:
    using Fibonacci = example_interfaces::action::Fibonacci;
    using GoalHandleFibonacci = rclcpp_action::ServerGoalHandle<Fibonacci>;

    FibonacciActionServer() : Node("fibonacci_action_server")
    {
        action_server_ = rclcpp_action::create_server<Fibonacci>(
            this, "fibonacci",
            std::bind(&FibonacciActionServer::handle_goal, this, std::placeholders::_1, std::placeholders::_2),
            std::bind(&FibonacciActionServer::handle_cancel, this, std::placeholders::_1),
            std::bind(&FibonacciActionServer::handle_accepted, this, std::placeholders::_1));
    }

private:
    rclcpp_action::Server<Fibonacci>::SharedPtr action_server_;

    rclcpp_action::GoalResponse handle_goal(
        const rclcpp_action::GoalUUID &uuid,
        std::shared_ptr<const Fibonacci::Goal> goal)
    {
        RCLCPP_INFO(this->get_logger(), "Received goal order: %d", goal->order);
        return rclcpp_action::GoalResponse::ACCEPT_AND_EXECUTE;
    }

    rclcpp_action::CancelResponse handle_cancel(
        const std::shared_ptr<GoalHandleFibonacci> goal_handle)
    {
        RCLCPP_INFO(this->get_logger(), "Cancel request");
        return rclcpp_action::CancelResponse::ACCEPT;
    }

    void handle_accepted(const std::shared_ptr<GoalHandleFibonacci> goal_handle)
    {
        std::thread{std::bind(&FibonacciActionServer::execute, this, goal_handle)}.detach();
    }

    void execute(const std::shared_ptr<GoalHandleFibonacci> goal_handle)
    {
        RCLCPP_INFO(this->get_logger(), "Executing goal");
        auto feedback = std::make_shared<Fibonacci::Feedback>();
        auto result = std::make_shared<Fibonacci::Result>();

        feedback->partial_sequence.push_back(0);
        feedback->partial_sequence.push_back(1);
        result->sequence = feedback->partial_sequence;

        goal_handle->publish_feedback(feedback);
        goal_handle->succeed(result);
    }
};
```

---

### 3.2 动作客户端

```cpp
#include "rclcpp/rclcpp.hpp"
#include "rclcpp_action/rclcpp_action.hpp"
#include "example_interfaces/action/fibonacci.hpp"

class FibonacciActionClient : public rclcpp::Node
{
public:
    using Fibonacci = example_interfaces::action::Fibonacci;
    using GoalHandleFibonacci = rclcpp_action::ClientGoalHandle<Fibonacci>;

    FibonacciActionClient() : Node("fibonacci_action_client")
    {
        action_client_ = rclcpp_action::create_client<Fibonacci>(this, "fibonacci");
    }

    void send_goal(int order)
    {
        if (!action_client_->wait_for_action_server()) {
            RCLCPP_ERROR(this->get_logger(), "Action server not available");
            return;
        }

        auto goal = Fibonacci::Goal();
        goal.order = order;

        auto send_goal_options = rclcpp_action::Client<Fibonacci>::SendGoalOptions();
        send_goal_options.feedback_callback = [](
            GoalHandleFibonacci::SharedPtr goal_handle,
            const std::shared_ptr<const Fibonacci::Feedback> feedback)
        {
            RCLCPP_INFO(rclcpp::get_logger("fibonacci_client"),
                "Feedback: %d", feedback->partial_sequence.back());
        };

        send_goal_options.result_callback = [](const GoalHandleFibonacci::WrappedResult &result)
        {
            switch (result.code) {
                case rclcpp_action::ResultCode::SUCCEEDED:
                    break;
                case rclcpp_action::ResultCode::ABORTED:
                    RCLCPP_ERROR(rclcpp::get_logger("fibonacci_client"), "Goal aborted");
                    break;
                case rclcpp_action::ResultCode::CANCELED:
                    RCLCPP_ERROR(rclcpp::get_logger("fibonacci_client"), "Goal canceled");
                    break;
                default:
                    RCLCPP_ERROR(rclcpp::get_logger("fibonacci_client"), "Unknown result code");
                    break;
            }
        };

        action_client_->async_send_goal(goal, send_goal_options);
    }

private:
    rclcpp_action::Client<Fibonacci>::SharedPtr action_client_;
};
```

---

## 4. Python 动作通讯

### 4.1 动作服务端

```python
import rclpy
from rclpy.node import Node
from rclpy.action import ActionServer
from rclpy.action.server import GoalResponse, CancelResponse
from example_interfaces.action import Fibonacci

class FibonacciActionServer(Node):
    def __init__(self):
        super().__init__('fibonacci_action_server')
        self._action_server = ActionServer(
            self, Fibonacci, 'fibonacci',
            execute_callback=self.execute_callback)

    async def execute_callback(self, goal_handle):
        self.get_logger().info('Executing goal...')
        feedback = Fibonacci.Feedback()
        feedback.partial_sequence = [0, 1]

        for i in range(1, goal_handle.request.order):
            feedback.partial_sequence.append(
                feedback.partial_sequence[i] + feedback.partial_sequence[i-1])
            goal_handle.publish_feedback(feedback)

        goal_handle.succeed()
        result = Fibonacci.Result()
        result.sequence = feedback.partial_sequence
        return result
```

---

### 4.2 动作客户端

```python
import rclpy
from rclpy.action import ActionClient
from rclpy.node import Node
from example_interfaces.action import Fibonacci

class FibonacciActionClient(Node):
    def __init__(self):
        super().__init__('fibonacci_action_client')
        self._action_client = ActionClient(self, Fibonacci, 'fibonacci')

    def send_goal(self, order):
        goal = Fibonacci.Goal()
        goal.order = order

        self._action_client.wait_for_server()
        self._send_goal_future = self._action_client.send_goal_async(
            goal, feedback_callback=self.feedback_callback)
        self._send_goal_future.add_done_callback(self.goal_response_callback)

    def goal_response_callback(self, future):
        goal_handle = future.result()
        if not goal_handle.accepted:
            self.get_logger().info('Goal rejected')
            return
        self.get_logger().info('Goal accepted')
        self._result_future = goal_handle.get_result_async()
        self._result_future.add_done_callback(self.get_result_callback)

    def get_result_callback(self, future):
        result = future.result().result
        self.get_logger().info(f'Result: {result.sequence}')

    def feedback_callback(self, feedback_msg):
        feedback = feedback_msg.feedback
        self.get_logger().info(f'Feedback: {feedback.partial_sequence[-1]}')
```

---

## 5. 动作命令

```bash
# 列出所有动作
ros2 action list

# 查看动作类型
ros2 action info /action_name

# 发送动作目标
ros2 action send_goal /action_name <action_type> "<goal>"

# 发送带反馈的目标
ros2 action send_goal /fibonacci example_interfaces/action/Fibonacci "{order: 5}" --feedback
```

---

# ROS2 自定义接口消息

## 1. 接口类型

ROS2支持三种接口类型:

| 类型 | 说明 |
|------|------|
| msg | 消息类型 |
| srv | 服务类型 (请求+响应) |
| action | 动作类型 (目标+反馈+结果) |

### 接口存放位置

```
package/
├── msg/
│   └── MyMessage.msg
├── srv/
│   └── MyService.srv
└── action/
    └── MyAction.action
```

---

## 2. 消息(msg)定义

### 2.1 基本消息

```yaml
# 示例: Person.msg
string name
uint8 age
float64 height

# 数组
string[] names
float64[] scores

# 常量
int32 X = 10
int32 Y = 20
```

### 2.2 嵌套消息

```yaml
# 使用已定义的消息
geometry_msgs/Pose pose
std_msgs/ColorRGBA color
```

---

## 3. 服务(srv)定义

```yaml
# Request部分
---
# Response部分

# 示例: DetectPerson.srv
sensor_msgs/Image image
---
bool success
string message
Person[] persons
```

---

## 4. 动作(action)定义

```yaml
# Goal部分
---
# Result部分
---
# Feedback部分

# 示例: MoveBase.action
geometry_msgs/PoseStamped target_pose
---
geometry_msgs/PoseStamped final_pose
float32 distance
---
geometry_msgs/PoseStamped current_pose
float32 remaining_distance
```

---

## 5. 接口使用示例

### 5.1 在C++中使用

```cpp
#include "my_package/msg/my_message.hpp"

// 创建消息
my_package::msg::MyMessage msg;
msg.name = "Tom";
msg.age = 25;

// 发布
publisher->publish(msg);
```

### 5.2 在Python中使用

```python
from my_package.msg import MyMessage

msg = MyMessage()
msg.name = "Tom"
msg.age = 25
publisher.publish(msg)
```

---

## 6. 编译接口

```cmake
# CMakeLists.txt
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/MyMessage.msg"
  "srv/MyService.srv"
  "action/MyAction.action"
  DEPENDENCIES std_msgs geometry_msgs
)
```

```xml
<!-- package.xml -->
<build_depend>rosidl_default_generators</build_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```

---

# ROS2 参数服务案例

## 1. 参数服务概念

- **参数(Parameter)**: 节点配置用的键值对
- 可在运行时动态修改
- 支持多种数据类型

### 参数类型

| 类型 | 说明 |
|------|------|
| int | 整数 |
| double | 浮点数 |
| bool | 布尔值 |
| string | 字符串 |
| byte[] | 字节数组 |

---

## 2. 参数命令行操作

```bash
# 列出节点参数
ros2 param list /node_name

# 获取参数值
ros2 param get /node_name param_name

# 设置参数值
ros2 param set /node_name param_name value

# 导出参数到文件
ros2 param dump /node_name > params.yaml

# 从文件加载参数
ros2 param load /node_name params.yaml
```

---

## 3. C++ 参数使用

### 3.1 声明参数

```cpp
class MyNode : public rclcpp::Node
{
public:
    MyNode() : Node("my_node")
    {
        // 声明参数
        this->declare_parameter("my_param", "default_value");
        this->declare_parameter("my_int", 10);
        this->declare_parameter("my_double", 3.14);
    }

    void use_parameter()
    {
        // 获取参数
        std::string value;
        this->get_parameter("my_param", value);

        // 获取参数(带默认值)
        auto int_value = this->get_parameter_or("my_int", 0);
    }
};
```

---

### 3.2 参数监听器

```cpp
class ParameterNode : public rclcpp::Node
{
public:
    ParameterNode() : Node("parameter_node")
    {
        // 创建参数监听器
        param_callback_handle_ = this->add_on_set_parameters_callback(
            std::bind(&ParameterNode::param_callback, this, std::placeholders::_1));
    }

    rcl_interfaces::msg::SetParametersResult param_callback(
        const std::vector<rclcpp::Parameter> &params)
    {
        for (const auto &param : params) {
            if (param.get_name() == "my_param") {
                RCLCPP_INFO(this->get_logger(), "my_param changed to: %s",
                    param.as_string().c_str());
            }
        }
        rcl_interfaces::msg::SetParametersResult result;
        result.successful = true;
        return result;
    }

    OnSetParametersCallbackHandle::SharedPtr param_callback_handle_;
};
```

---

## 4. Python 参数使用

### 4.1 声明参数

```python
class MyNode(Node):
    def __init__(self):
        super().__init__('my_node')

        # 声明参数
        self.declare_parameter('my_param', 'default_value')
        self.declare_parameter('my_int', 10)
        self.declare_parameter('my_double', 3.14)

    def use_parameter(self):
        # 获取参数
        value = self.get_parameter('my_param').value

        # 获取参数(带默认值)
        int_value = self.get_parameter_or('my_int', 0)
```

---

### 4.2 参数回调

```python
class ParameterNode(Node):
    def __init__(self):
        super().__init__('parameter_node')

        # 添加参数回调
        self.add_on_set_parameters_callback(self.param_callback)

    def param_callback(self, params):
        for param in params:
            if param.name == 'my_param':
                self.get_logger().info(f'my_param changed to: {param.value}')
        return SetParametersResult(successful=True)
```

---

## 5. 参数文件示例

```yaml
my_node:
  ros__parameters:
    my_param: "hello"
    my_int: 42
    my_double: 3.14
    my_bool: true

    # 嵌套参数
    nested:
      param1: 1
      param2: 2
```

---

# ROS2 元功能包介绍

## 1. 元功能包概念

- **元功能包(Metapackage)**: 只包含依赖关系的空功能包
- 用于逻辑组织相关功能包
- 本身不包含代码

### 常见元功能包

| 元功能包 | 内容 |
|----------|------|
| ros-base | 核心功能 |
| desktop | 完整桌面工具 |
| demos | 示例代码 |

---

## 2. 创建元功能包

### 2.1 package.xml

```xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd"?>
<package format="3">
  <name>my_metapackage</name>
  <version>1.0.0</version>
  <description>My metapackage</description>
  <maintainer email="user@example.com">User</maintainer>
  <license>Apache-2.0</license>

  <export>
    <metapackage/>
  </export>

  <depend>package1</depend>
  <depend>package2</depend>
  <depend>package3</depend>
</package>
```

### 2.2 CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.8)
project(my_metapackage)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_index_cpp REQUIRED)

ament_package()
```

---

## 3. 使用元功能包

```bash
# 安装元功能包
sudo apt install ros-humble-desktop

# 查看元功能包内容
apt show ros-humble-desktop
```

---

# ROS2 分布式通讯

## 1. 分布式通讯概念

- ROS2支持多机器分布式部署
- 基于DDS实现跨机器通信
- 需要网络配置

### 应用场景

- 多机器人系统
- 仿真与实际结合
- 分布式计算

---

## 2. 网络配置

### 2.1 环境变量

```bash
# 设置ROS_DOMAIN_ID
export ROS_DOMAIN_ID=42

# 同一网络下设备使用相同ID
echo "export ROS_DOMAIN_ID=42" >> ~/.bashrc
```

### 2.2 主机网络配置

```bash
# 查看IP地址
hostname -I
ip addr show

# 测试网络连通性
ping <other_machine_ip>
```

---

## 3. 跨机器通信设置

### 3.1 主机A配置

```bash
# 设置环境变量
export ROS_DOMAIN_ID=1
export ROS_LOCALHOST_ONLY=0

# 启动节点
ros2 run demo_nodes_cpp listener
```

### 3.2 主机B配置

```bash
# 设置相同DOMAIN_ID
export ROS_DOMAIN_ID=1

# 启动发布者
ros2 run demo_nodes_cpp talker
```

---

## 4. 防火墙配置

```bash
# Ubuntu防火墙
sudo ufw allow 7400/udp  # DDS端口
sudo ufw allow 7401/udp

# 或直接关闭(测试环境)
sudo ufw disable
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

# ROS2 时间相关API

## 1. 时间类型

### 1.1 Duration

```cpp
#include "rclcpp/time.hpp"

// 创建Duration
rclcpp::Duration duration(1s);           // 1秒
rclcpp::Duration duration(1, 500000000);  // 1.5秒

// Duration运算
auto sum = duration1 + duration2;
auto diff = duration1 - duration2;
```

### 1.2 Time

```cpp
// 获取当前时间
rclcpp::Time now = this->now();

// 创建指定时间
rclcpp::Time time1(1, 0);  // 1秒

// 时间运算
auto later = now + duration;
auto earlier = now - duration;
```

---

## 2. 定时器

### 2.1 C++ 定时器

```cpp
class TimerNode : public rclcpp::Node
{
public:
    TimerNode() : Node("timer_node")
    {
        // 创建定时器 (1秒周期)
        timer_ = this->create_wall_timer(
            1s, std::bind(&TimerNode::timer_callback, this));
    }

private:
    void timer_callback()
    {
        auto now = this->now();
        RCLCPP_INFO(this->get_logger(), "Timer triggered at: %.2f", now.nanoseconds() / 1e9);
    }

    rclcpp::TimerBase::SharedPtr timer_;
};
```

---

### 2.2 Python 定时器

```python
class TimerNode(Node):
    def __init__(self):
        super().__init__('timer_node')

        # 创建定时器 (1秒周期)
        self.timer = self.create_timer(1.0, self.timer_callback)

    def timer_callback(self):
        now = self.get_clock().now()
        self.get_logger().info(f'Timer triggered at: {now.nanoseconds() / 1e9}')
```

---

## 3. 等待时间

### 3.1 C++ 等待

```cpp
// 等待一段时间
std::this_thread::sleep_for(1s);

// 等待条件
std::condition_variable cv;
std::mutex mtx;
std::unique_lock<std::mutex> lock(mtx);
cv.wait_for(lock, 1s);

// 使用rclcpp等待
rclcpp::Duration timeout(1s);
std::this_thread::sleep_for(std::chrono::nanoseconds(timeout.nanoseconds()));
```

### 3.2 Python 等待

```python
import time

# 等待
time.sleep(1)  # 秒

# 带条件的等待
import threading
event = threading.Event()
event.wait(timeout=1)
```

---

## 4. 时间转换

```cpp
// Duration转换
auto seconds = duration.seconds();
auto nanoseconds = duration.nanoseconds();

// Time转换
auto time_sec = time_obj.seconds();

// 与ROS1时间转换
rcl_time_point_value_t time_val;
time_obj = rclcpp::Time(time_val);

// 获取系统时间
std::chrono::system_clock::time_point now = std::chrono::system_clock::now();
auto epoch = now.time_since_epoch();
auto seconds = std::chrono::duration_cast<std::chrono::seconds>(epoch).count();
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

---

# The Transform Library

## 1. Overview

- TF2 is a transformative library for robotics
- Handles coordinate transformations seamlessly
- Supports multiple coordinate frames over time

---

## 2. Key Concepts

### 2.1 Coordinate Frames

- World frame (origin)
- Robot base frame
- Sensor frames
- End-effector frames

### 2.2 Time-based Transforms

- Transforms vary over time
- Buffer stores historical transforms
- Interpolation for smooth transformations

---

## 3. Implementation

### 3.1 Basic Transform

```cpp
// Create transform
geometry_msgs::msg::TransformStamped transform;
transform.header.stamp = now();
transform.header.frame_id = "base_link";
transform.child_frame_id = "laser";
transform.transform.translation.x = 0.2;
```

### 3.2 Transform Types

- Static transforms (constant)
- Dynamic transforms (time-varying)
- Kinematic chains

---

## 4. Best Practices

- Use meaningful frame names
- Publish transforms at consistent rates
- Minimize transform tree depth
- Use static transforms when appropriate
- Handle transform exceptions gracefully
