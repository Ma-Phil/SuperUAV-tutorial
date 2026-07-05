# 启动脚本与启动顺序

本页整理当前 Super 机 NUC 软件启动顺序。初次调试时先分终端启动每个节点，确认主链路都能单独工作；全部通过后，再使用整合启动脚本。

主链路顺序为：

```text
roscore
-> MAVROS
-> livox_ros_driver2 / msg_MID360.launch
-> Point-LIO-new
-> RealSense 可选
```

## 分步启动主链路

每个终端启动节点前，按需要加载对应工作空间：

```bash
source /opt/ros/noetic/setup.bash
source ~/rosprojects/init_mav/livox_ws/devel/setup.bash
source ~/rosprojects/init_mav/lio_ws/devel/setup.bash
source ~/rosprojects/init_mav/mavros_ws/devel/setup.bash
```

先启动 ROS master：

```bash
roscore
```

再启动 PX6C USB MAVROS：

```bash
roslaunch mavros px4.launch fcu_url:=/dev/ttyACM0:921600
```

然后启动 Mid-360 自定义消息：

```bash
roslaunch livox_ros_driver2 msg_MID360.launch
```

再启动 Point-LIO-new：

```bash
roslaunch point_lio mapping_avia.launch rviz:=false
```

Point-LIO 到 `/mavros/vision_pose/pose` 的桥接配置与验证见 [Point-LIO-new 安装与调参](point-lio.md#point-lio-mavros-vision-pose-bridge)。整合启动时，桥接节点应放在 Point-LIO-new 启动之后、EKF 检查之前。

RealSense 为可选外设；未安装时跳过：

```bash
roslaunch realsense2_camera rs_camera.launch
```

!!! note "Point-LIO launch 名称"
    当前记录中 Point-LIO-new 仍使用 `mapping_avia.launch`。实际操作时应以 `~/rosprojects/init_mav/lio_ws/src/point_lio_new` 中存在的 launch 文件为准；若后续有专门的 Mid-360 launch，应同步替换。

## 最小通过标准

- `/mavros/state` 显示 `connected: True`。
- `/livox/lidar` 或 driver2 对应话题有稳定频率。
- Point-LIO 持续输出位姿 / 里程计。
- `/mavros/vision_pose/pose` 有稳定频率。
- `/mavros/local_position/odom` 在外部定位启动后稳定输出，并确认话题格式对齐。

## 整合启动脚本

下面的脚本用于分步验证通过后的整合启动。桥接节点必须插在 Point-LIO-new 启动之后、EKF 检查之前；桥接配置见 [Point-LIO-new 安装与调参](point-lio.md#point-lio-mavros-vision-pose-bridge)，这里仅保留插入位置，不新增具体桥接源码。

```bash
#!/usr/bin/env bash

# 加载系统 ROS
source /opt/ros/noetic/setup.bash

# 加载 Livox ROS Driver 2
source ~/rosprojects/init_mav/livox_ws/devel/setup.bash --extend

# 加载 Point-LIO-new
source ~/rosprojects/init_mav/lio_ws/devel/setup.bash --extend

# 加载 MAVROS
source ~/rosprojects/init_mav/mavros_ws/devel/setup.bash --extend

# RealSense 可选；如果未安装则跳过
if [ -f ~/rosprojects/init_mav/realsense_ros_ws/devel/setup.bash ]; then
  source ~/rosprojects/init_mav/realsense_ros_ws/devel/setup.bash --extend
fi

# 启动 roscore
roscore &
sleep 2

# 启动 PX6C USB MAVROS
roslaunch mavros px4.launch fcu_url:="/dev/ttyACM0:921600" &
sleep 3

# 启动 Mid-360 雷达自定义消息
roslaunch livox_ros_driver2 msg_MID360.launch &
sleep 1

# 启动 Point-LIO-new
roslaunch point_lio mapping_avia.launch rviz:=false &
sleep 2

# 在这里启动 Point-LIO -> /mavros/vision_pose/pose 桥接节点
# roslaunch <your_bridge_package> <your_bridge.launch> &

# RealSense 可选；如果未安装 RealSense，注释掉这一行
roslaunch realsense2_camera rs_camera.launch &

wait
```
