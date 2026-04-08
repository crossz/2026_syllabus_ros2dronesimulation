
# 🧭 教学目标

用 **QGroundControl** 作为统一可视化入口：

| 系统       | 数据来源          | 中间层                         | 输出到 QGC  |
| -------- | ------------- | --------------------------- | -------- |
| 无人机（PX4） | gz sim camera | PX4 SITL + GStreamer plugin | UDP H264 |
| 小车（ROS2） | gz sim camera | ROS2 + GStreamer node       | UDP H264 |

👉 **关键统一点：**

> 不管无人机还是小车，最后都变成
> 👉 **“H264 over UDP → QGC”**

---

# 一、统一环境（Ubuntu 24.04 + Jazzy + Harmonic）

## 1️⃣ 安装 Gazebo Harmonic（gz sim v8）

```bash
sudo apt update
sudo apt install -y gz-harmonic
```

验证：

```bash
gz sim --version
# 应该是 gz sim 8.x
```

👉 对应：Gazebo Harmonic

---

## 2️⃣ 安装 ROS2 Jazzy

👉 对应：ROS 2 Jazzy Jalisco

```bash
sudo apt install ros-jazzy-desktop
source /opt/ros/jazzy/setup.bash
```

---

## 3️⃣ 安装 ros_gz bridge

```bash
sudo apt install ros-jazzy-ros-gz
```

👉 对应：ros_gz

---

## 4️⃣ 安装 GStreamer（两套实验通用）

```bash
sudo apt install -y \
  gstreamer1.0-tools \
  gstreamer1.0-plugins-base \
  gstreamer1.0-plugins-good \
  gstreamer1.0-plugins-bad \
  gstreamer1.0-plugins-ugly \
  gstreamer1.0-libav \
  python3-gi
```

---

# 二、实验 A（无人机）

## PX4 + gz sim v8 → QGC 视频

👉 对应：PX4 Autopilot

---

## ⚠️ 关键说明（非常重要）

在 **Gazebo Harmonic（gz sim）** 中：

👉 PX4 **默认不会自动推视频到 QGC**（和 Gazebo Classic 不同）

所以必须：

👉 **自己加一个 GStreamer pipeline**

---

## 1️⃣ 启动 PX4 SITL（gz sim）

```bash
cd ~/PX4-Autopilot
make px4_sitl gz_x500
```

👉 对应：PX4 SITL

---

## 2️⃣ 找到 camera topic（gz）

新开终端：

```bash
gz topic -l
```

你会看到类似：

```
/world/default/model/x500_0/link/camera_link/sensor/camera/image
```

---

## 3️⃣ 用 gz → GStreamer 直接推流（关键步骤）

```bash
gz topic -e -t /world/default/model/x500_0/link/camera_link/sensor/camera/image \
| gst-launch-1.0 fdsrc ! \
  video/x-raw,format=RGB,width=640,height=480 ! \
  videoconvert ! \
  x264enc tune=zerolatency bitrate=800 speed-preset=ultrafast ! \
  rtph264pay ! \
  udpsink host=127.0.0.1 port=5600
```

👉 这是 **无人机版本的“桥接”**

---

## 4️⃣ QGC 设置

* Video Source: UDP h264
* Port: **5600**

👉 打开视频 → 成功 🎉

---

## 🎯 教学讲解点（无人机）

👉 这里其实没有 ROS2：

```
gz sim → (手动 pipeline) → QGC
```

👉 强调：

* PX4 不处理视频
* Gazebo 提供图像
* 我们人为接管“视频链路”

---

# 三、实验 B（ROS2 小车）

## gz sim + ROS2 Jazzy → QGC

---

## 1️⃣ 启动 gz sim 小车

```bash
gz sim shapes.sdf
```

或你自己的 robot world

---

## 2️⃣ 启动 ROS2 bridge

```bash
ros2 run ros_gz_bridge parameter_bridge \
/camera@sensor_msgs/msg/Image@gz.msgs.Image
```

---

## 3️⃣ ROS2 → GStreamer 节点（核心）

👉 这里直接给你 **Jazzy 可用版本（已优化）**

```python
# ros2_gst_streamer_jazzy.py

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from cv_bridge import CvBridge
import gi
gi.require_version('Gst', '1.0')
from gi.repository import Gst

Gst.init(None)

class Streamer(Node):
    def __init__(self):
        super().__init__('streamer')
        self.bridge = CvBridge()

        self.sub = self.create_subscription(
            Image, '/camera', self.cb, 10)

        self.pipeline = Gst.parse_launch(
            'appsrc name=src is-live=true format=TIME '
            '! videoconvert '
            '! x264enc tune=zerolatency bitrate=800 speed-preset=ultrafast '
            '! rtph264pay '
            '! udpsink host=127.0.0.1 port=5602'
        )

        self.appsrc = self.pipeline.get_by_name('src')
        self.pipeline.set_state(Gst.State.PLAYING)

    def cb(self, msg):
        frame = self.bridge.imgmsg_to_cv2(msg, 'bgr8')
        data = frame.tobytes()

        buf = Gst.Buffer.new_allocate(None, len(data), None)
        buf.fill(0, data)

        self.appsrc.emit('push-buffer', buf)

def main():
    rclpy.init()
    node = Streamer()
    rclpy.spin(node)

if __name__ == '__main__':
    main()
```

运行：

```bash
python3 ros2_gst_streamer_jazzy.py
```

---

## 4️⃣ QGC 设置

* Port: **5602**

---

# 四、核心对比（你课堂最重要的一页）

## 🧠 架构对比图

### 🚁 PX4（无人机）

```
gz sim camera
   ↓
gz topic
   ↓
GStreamer pipeline（手写）
   ↓
UDP 5600
   ↓
QGC
```

---

### 🚗 ROS2 小车

```
gz sim camera
   ↓
ros_gz bridge
   ↓
ROS2 topic
   ↓
ROS2 node + GStreamer
   ↓
UDP 5602
   ↓
QGC
```

---

## 🎯 一句话总结（课堂重点）

👉 **无人机 = 模拟器直出视频**
👉 **小车 = ROS2加工后输出视频**

但：

👉 **QGC 完全不知道区别**

---

# 五、你可以强调的“高级理解点”（非常加分）

### 1️⃣ QGC 本质

👉 只是一个：

> 🎥 **H264 视频播放器 + MAVLink 客户端**

---

### 2️⃣ MAVLink Camera vs Video Stream

👉 控制（拍照/录像）≠ 视频流

---

### 3️⃣ 为什么统一用 QGC？

* 不用 RViz2（降低复杂度）
* 和真实无人机一致
* 学生理解“真实系统”

---

# 六、推荐视频（强烈建议课前看）

---

# 七、如果你要做“教学升级版”

我可以帮你做👇（直接可上课用）：

### ✅ 1. PPT（最推荐）

包含：

* 架构对比图（非常清晰）
* 数据流动画
* 常见错误演示

---

### ✅ 2. 一键脚本

```bash
./start_px4_video.sh
./start_ros2_video.sh
```

---

### ✅ 3. Docker 版本（统一环境）

避免学生环境问题

---

### ✅ 4. 双窗口对比演示（很震撼）

左：无人机
右：小车
都在 QGC
