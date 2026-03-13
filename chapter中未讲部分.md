
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

----


## QGroundControl | QGC 连接

如果要在windows上的 QGC链接虚拟机中的 gazebo，需要设置环境变量

```bash
export PX4_SIM_HOST_ADDR=192.168.1.101
make px4_sitl gz_x500
```

如此，不必需要在同一个linux实例中运行 QGC（和 gazebo一起）