
## 关闭 Terminal（终端）中运行的程序

✅：在 termial 中通过 `ctrl + c` 快捷键结束运行中的命令行程序

适用于：
- `ros2 launch`
- `ros2 run`
- px4 模拟器：`make px4_sitl gz_x500` 和相关的命令行
- QGroundControl：`./QGroundControl.AppImage`
- 其他：待补充

❌：直接关闭 termial 的窗口

## gazebo 进程的强制关闭（gazebo 不正常，优先排查此项）

查看当前 Linux 操作系统中是否后台还有 gazebo(新版程序名为 gz) 的进程
✅：
```bash
ps aux | grep gz
```
输出内容：
```bash
ubuntu@ubuntu-VMware:~/PX4-Autopilot$ ps aux | grep gz
ubuntu      3650 46.0  2.4 1647384 195952 ?      Sl   20:41   0:23 gz sim --verbose=1 -r -s /home/ubuntu/PX4-Autopilot/Tools/simulation/gz/worlds/default.sdf
ubuntu      3651  135 10.6 3493916 859720 ?      Sl   20:41   1:08 gz sim -g
ubuntu      4259  0.0  0.0   9152  2328 pts/1    S+   20:42   0:00 grep --color=auto gz
```

除了最后一行 `grep`(搜索的意思) 的进程，这是 `ps aux | grep gz` 本身自己进程，其他进程都要强制关闭（kill）。将输出内容中每行的第 2 列，对应进程的 id，作为 kill 的参数，用以关闭指定的进程。其中 -9 是强制的信号（没有 -9）那么系统会一直等合适的时间再关闭。

✅：`kill -9 xxxx xxxx 
> 上面例子中 xxxx 替换进程id，则应该是 `kill -9 3650 3651`

---

## Linux 的 terminal 基本操作

### 查看当前文件夹内容

```bash
ls
```
输出内容：
```bash
ubuntu@ubuntu-VMware:~$ ls
Desktop    Micro-XRCE-DDS-Agent  Public                   Resources  Templates
Documents  Music                 PX4-Autopilot            ros2_ws    Videos
Downloads  Pictures              QGroundControl.AppImage  snap
```

### 切换到指定文件夹内，比如 `PX4-Autopilot`

```bash
cd PX4-Autopilot
```
输出内容：
```bash
ubuntu@ubuntu-VMware:~$ cd PX4-Autopilot/
ubuntu@ubuntu-VMware:~/PX4-Autopilot$ 
ubuntu@ubuntu-VMware:~/PX4-Autopilot$ ls
boards              docs              MAINTAINERS.md  README.md    test_data
build               integrationtests  Makefile        ROMFS        Tools
cmake               Jenkinsfile       msg             SECURITY.md  validation
CMakeLists.txt      Kconfig           package.xml     src
CODE_OF_CONDUCT.md  launch            platforms       srv
CONTRIBUTING.md     LICENSE           posix-configs   test
```