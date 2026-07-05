# 三、NUC 配置

本部分面向机载 NUC，固定系统为 **Ubuntu 20.04 + ROS Noetic**。当前 Super 无人机的软件链路固定为：

```text
Livox Mid-360
        -> livox_ros_driver2 / msg_MID360.launch
        -> NKU-UAVTeam/Point-LIO-new
        -> MAVROS /dev/ttyACM0:921600
        -> /mavros/vision_pose/pose
        -> PX4 v1.13.3 EKF2 融合 vision position + vision yaw
```

RealSense 是可选外设，不作为主定位融合链路。

!!! note "NUC 主线顺序"
    先完成基础依赖、统一工作空间和固定源码仓库安装；这些前置条件确认后，再配置 Mid-360 雷达 IP、MAVROS、Point-LIO-new 调参、启动脚本和 EKF 检查。

## 基础依赖

如果 NUC 还没有基础工具和 ROS 依赖，先安装：

```bash
sudo apt update
sudo apt install -y git build-essential cmake wget curl
sudo apt install -y python3-catkin-tools python3-rosdep python3-vcstool python3-wstool
sudo apt install -y libeigen3-dev libpcl-dev
sudo apt install -y ros-noetic-pcl-conversions ros-noetic-pcl-ros
sudo apt install -y ros-noetic-tf ros-noetic-tf2-ros ros-noetic-tf2-eigen
```

ROS Noetic 初始化：

```bash
sudo rosdep init
rosdep update
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

如果 `sudo rosdep init` 提示已经初始化，只执行 `rosdep update`。

## 工作空间约定

所有 NUC 侧程序统一放在 `~/rosprojects/init_mav/` 下。后续教程和启动脚本均按以下路径书写：

```text
~/rosprojects/init_mav/
~/rosprojects/init_mav/livox_SDK
~/rosprojects/init_mav/livox_ws
~/rosprojects/init_mav/livox_ws/src/livox_ros_driver2
~/rosprojects/init_mav/lio_ws
~/rosprojects/init_mav/lio_ws/src/point_lio_new
~/rosprojects/init_mav/mavros_ws
~/rosprojects/init_mav/mavros_ws/src/mavros
~/rosprojects/init_mav/realsense_SDK
~/rosprojects/init_mav/realsense_ros_ws
~/rosprojects/init_mav/realsense_ros_ws/src/realsense-ros
```

先创建基础目录：

```bash
mkdir -p ~/rosprojects/init_mav
```

如果实际目录不同，需要同步修改 [启动脚本](startup.md) 中所有 `source` 路径。

## 固定源码仓库（先安装）

下表不是参考链接，而是 NUC 配置前需要安装或克隆到对应工作空间的源码。先完成这些软件安装，再进入 Mid-360 雷达 IP、广播码和 `MID360_config.json` 配置。

| 模块 | 固定仓库 / 版本 | 安装要求 | 固定路径 | 用途 |
| --- | --- | --- | --- | --- |
| Livox SDK2 | [Livox-SDK/Livox-SDK2](https://github.com/Livox-SDK/Livox-SDK2) | 必装 | `~/rosprojects/init_mav/livox_SDK` | Mid-360 底层 SDK。 |
| Livox ROS Driver 2 | [Livox-SDK/livox_ros_driver2](https://github.com/Livox-SDK/livox_ros_driver2) | 必装 | `~/rosprojects/init_mav/livox_ws/src/livox_ros_driver2` | ROS1 驱动，启动 `msg_MID360.launch`。 |
| Point-LIO-new | [NKU-UAVTeam/Point-LIO-new](https://github.com/NKU-UAVTeam/Point-LIO-new.git) | 必装；私有仓库，需先加入 GitHub Team，具体询问林毓文 | `~/rosprojects/init_mav/lio_ws/src/point_lio_new` | 本机使用的 Point-LIO 版本。 |
| MAVROS | [mavlink/mavros, master](https://github.com/mavlink/mavros/tree/master) | 必装 | `~/rosprojects/init_mav/mavros_ws/src/mavros` | NUC 与 PX4 飞控通信。 |
| librealsense | [IntelRealSense/librealsense v2.50.0](https://github.com/IntelRealSense/librealsense) | 可选 | `~/rosprojects/init_mav/realsense_SDK` | 可选 RealSense SDK。 |
| realsense-ros | [IntelRealSense/realsense-ros 2.3.2](https://github.com/IntelRealSense/realsense-ros) | 可选 | `~/rosprojects/init_mav/realsense_ros_ws/src/realsense-ros` | 可选 ROS1 wrapper。 |

## 推荐配置顺序

1. 完成基础依赖和 `~/rosprojects/init_mav/` 目录创建。
2. [Mid-360 雷达配置](mid360.md)：安装 Livox SDK2、livox_ros_driver2，并配置雷达 IP / `MID360_config.json`。
3. [Point-LIO-new 安装与调参](point-lio.md)：确认私有仓库权限，安装 Point-LIO-new，检查 `mapping_avia.launch` / `avia.yaml`，并准备 `vision_pose` 桥接。
4. [PX6C USB 连接 MAVROS](px6c-usb-mavros.md)：安装 MAVROS，配置 USB 权限、确认 `/dev/ttyACM0`、检查 MAVROS 连接。
5. [启动脚本与启动顺序](startup.md)：先分步启动 `roscore`、MAVROS、Livox driver2、Point-LIO，再整合成启动脚本。
6. 确认 `vision_pose` 桥接输出后，进入 [EKF Gate 与融合检查](../flight-debug/ekf-gate.md)。
7. 如需要相机，再配置 [RealSense 可选配置](realsense.md)。
