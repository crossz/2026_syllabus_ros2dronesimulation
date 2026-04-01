- 切入 claude code 或者 opencode

- turtulebot3 的 rviz2 的实操
- SLAM 和 nav2 的实操
- 插入 fuel 的 model 和 world


---
## PX4
#### 参考资源
官网：
1. [综述](https://docs.px4.io/main/en/sim_gazebo_gz/)
2. [实战](https://docs.px4.io/main/en/sim_gazebo_gz/)
3. [模拟器（社区支持的）](https://docs.px4.io/main/en/simulation/community_supported_simulators)

阿木实验室：
1. [商业开源的一个版本（没有完全升级完成）](https://docs.amovlab.com/prometheus-wiki/#/src/PrometheusV3%E5%85%88%E9%A9%B1%E8%80%85/PrometheusV3_DDS%E9%83%A8%E7%BD%B2/3.%E9%83%A8%E7%BD%B2PrometheusV3)
2. [其在知乎上发的文章](https://zhuanlan.zhihu.com/p/1924402597852324462)



#### 起飞
```bash
export PX4_HOME_LAT=25.926562 
export PX4_HOME_LON=119.493454 
export PX4_HOME_ALT=15
make px4_sitl gz_x500
```
