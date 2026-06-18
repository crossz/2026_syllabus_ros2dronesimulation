https://monica.im/share/chat?shareId=Rq35AxGbuMwTABfW

# 🐢 TurtleBot4 在 ROS2 Jazzy + Gazebo Harmonic (gz sim v8) 中运行教程

采用 lite 模式，比较简单


> **环境要求：** Ubuntu 24.04 LTS + ROS2 Jazzy + Gazebo Harmonic (gz sim 8)

---

## 第一步：安装 ROS2 Jazzy 基础环境

```bash
# 设置 locale
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8

# 添加 ROS2 源
sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
  http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | \
  sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

# 安装 ROS2 Jazzy Desktop
sudo apt update
sudo apt install ros-jazzy-desktop
```

[1]: https://docs.ros.org/en/jazzy/Tutorials/Advanced/Simulators/Gazebo/Simulation-Gazebo.html

---

## 第二步：安装 Gazebo Harmonic (gz sim v8)

```bash
# 添加 Gazebo 源
sudo apt-get update
sudo apt-get install curl lsb-release gnupg

sudo curl https://packages.osrfoundation.org/gazebo.gpg \
  --output /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] \
  http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null

# 安装 Gazebo Harmonic
sudo apt-get update
sudo apt-get install gz-harmonic

# 安装 ROS2-Gazebo 桥接包
sudo apt install ros-jazzy-ros-gz
```

[1]: https://docs.ros.org/en/jazzy/Tutorials/Advanced/Simulators/Gazebo/Simulation-Gazebo.html

---

## 第三步：安装 TurtleBot4 仿真包

这是最关键的一步。在 Jazzy 版本中，TurtleBot4 已将所有包名从 `ignition/ign` 重命名为 `gz` 命名规范。[3]: https://turtlebot.github.io/turtlebot4-user-manual/changelogs/jazzy.html

```bash
# 安装 TurtleBot4 仿真器元包（推荐方式）
sudo apt update
sudo apt install ros-jazzy-turtlebot4-simulator

# 安装相关依赖
sudo apt install \
  ros-jazzy-turtlebot4-gz-gui-plugins \
  ros-jazzy-irobot-create-nodes \
  ros-jazzy-irobot-create-gz-plugins \
  ros-jazzy-ros-gz
```

> ⚠️ **注意：** 在 Jazzy 中，仿真包已从 `turtlebot4_ignition_bringup` 迁移到 `turtlebot4_gz_bringup`，请勿使用旧的 ignition 包。

[2]: https://turtlebot.github.io/turtlebot4-user-manual/software/turtlebot4_simulator.html

---

## 第四步：配置环境变量

```bash
# 添加到 ~/.bashrc
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

---

## 第五步：启动 TurtleBot4 仿真

### ✅ 启动标准版 TurtleBot4（默认世界）

```bash
ros2 launch turtlebot4_gz_bringup turtlebot4_gz.launch.py
```

### ✅ 启动 TurtleBot4 Lite 版本

```bash
ros2 launch turtlebot4_gz_bringup turtlebot4_gz.launch.py model:=lite
```

### ✅ 常用可选参数

| 参数 | 说明 | 示例值 |
|------|------|--------|
| `model` | 机器人型号 | `standard`（默认）/ `lite` |
| `world` | 仿真世界 | `maze` / `depot` / `warehouse` |
| `namespace` | 机器人命名空间 | `robot1` |
| `x` / `y` / `z` | 初始位置 | `0.0` |

### ✅ 指定世界启动示例

```bash
# 在 maze 迷宫世界中启动
ros2 launch turtlebot4_gz_bringup turtlebot4_gz.launch.py world:=maze

# 在 depot 仓库世界中启动
ros2 launch turtlebot4_gz_bringup turtlebot4_gz.launch.py world:=depot
```

[4]: https://turtlebot.github.io/turtlebot4-user-manual/software/simulation.html

---

## 第六步：验证仿真运行

启动成功后，在新终端中验证话题：

```bash
# source 环境
source /opt/ros/jazzy/setup.bash

# 查看所有话题
ros2 topic list

# 查看摄像头图像话题
ros2 topic echo /oakd/rgb/preview/image_raw

# 查看激光雷达数据
ros2 topic echo /scan

# 查看里程计
ros2 topic echo /odom
```

### 手动控制机器人（键盘）

```bash
# 安装 teleop 包
sudo apt install ros-jazzy-teleop-twist-keyboard

# 启动键盘控制
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

---

## ⚠️ 常见问题排查

### 问题1：插件找不到（Plugin not found）

如果出现类似错误：
```
[Err] Failed to load system plugin [libgazebo_ros_create_wheel_drop.so]
```

**原因：** 使用了旧的 `gazebo_ros` 插件，而非新的 `gz` 插件。  
**解决：** 确保安装的是 `ros-jazzy-irobot-create-gz-plugins` 而非旧版 ignition 插件。[5]: https://robotics.stackexchange.com/questions/114380/how-to-spawn-turtlebot-4-urdf-with-ros2-jazzy-in-gazebo-harmonic

```bash
sudo apt install ros-jazzy-irobot-create-gz-plugins
sudo apt install ros-jazzy-turtlebot4-gz-gui-plugins
```

### 问题2：包名混淆（ignition vs gz）

在 Jazzy 版本中，所有包已重命名：

| 旧名（Humble/Iron） | 新名（Jazzy） |
|---|---|
| `turtlebot4_ignition_bringup` | `turtlebot4_gz_bringup` |
| `turtlebot4_ignition_toolbox` | `turtlebot4_gz_toolbox` |
| `ros_ign_bridge` | `ros_gz_bridge` |

[3]: https://turtlebot.github.io/turtlebot4-user-manual/changelogs/jazzy.html

---

## 📚 参考资料

| #   | 资料                       | 链接                                                                                        |
| --- | ------------------------ | ----------------------------------------------------------------------------------------- |
| [1] | ROS2 Jazzy + Gazebo 官方文档 | https://docs.ros.org/en/jazzy/Tutorials/Advanced/Simulators/Gazebo/Simulation-Gazebo.html |
| [2] | TurtleBot4 仿真器安装手册       | https://turtlebot.github.io/turtlebot4-user-manual/software/turtlebot4_simulator.html     |
| [3] | TurtleBot4 Jazzy 版本变更日志  | https://turtlebot.github.io/turtlebot4-user-manual/changelogs/jazzy.html                  |
| [4] | TurtleBot4 仿真启动说明        | https://turtlebot.github.io/turtlebot4-user-manual/software/simulation.html               |
| [5] | Jazzy + Harmonic 插件问题排查  | https://robotics.stackexchange.com/questions/114380                                       |
