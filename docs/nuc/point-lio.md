# Point-LIO-new 安装与调参

本页用于安装当前 Super 机使用的 **Point-LIO-new**，并记录 `mapping_avia.launch` 与 `avia.yaml` 中需要重点检查的调参项。Point-LIO-new 依赖 Livox ROS Driver 2，因此先完成 [Mid-360 雷达配置](mid360.md) 中的 Livox SDK2 和 livox_ros_driver2 安装。

Point-LIO-new 安装是主定位链路必须完成的步骤；后面的调参内容仅供雷达定位输出异常、漂移或初始化问题时排查参考，正常安装验证时不需要逐项改参数。

固定路径：

```text
~/rosprojects/init_mav/lio_ws/src/point_lio_new
```

!!! warning "私有仓库权限"
    `NKU-UAVTeam/Point-LIO-new.git` 是私有仓库。克隆前 GitHub 账号需要先加入对应 Team；权限开通和账号问题具体询问林毓文。

    如果 `git clone` 提示 `Repository not found`、认证失败或无权限，先处理 GitHub Team 权限，再继续安装。

## 安装 Point-LIO-new

先加载 ROS 和 livox_ros_driver2 工作空间：

```bash
source /opt/ros/noetic/setup.bash
source ~/rosprojects/init_mav/livox_ws/devel/setup.bash
```

创建工作空间并克隆源码：

```bash
mkdir -p ~/rosprojects/init_mav/lio_ws/src
cd ~/rosprojects/init_mav/lio_ws/src
git clone https://github.com/NKU-UAVTeam/Point-LIO-new.git point_lio_new
cd point_lio_new
git submodule update --init
```

编译：

```bash
cd ~/rosprojects/init_mav/lio_ws
source /opt/ros/noetic/setup.bash
source ~/rosprojects/init_mav/livox_ws/devel/setup.bash
catkin_make
source devel/setup.bash
```

安装完成后确认包可以被 ROS 找到：

```bash
rospack find point_lio
```

## 启动方式

当前教程保留 Point-LIO README 中的主启动方式：

```bash
source /opt/ros/noetic/setup.bash
source ~/rosprojects/init_mav/livox_ws/devel/setup.bash
source ~/rosprojects/init_mav/lio_ws/devel/setup.bash
roslaunch point_lio mapping_avia.launch rviz:=false
```

启动前应确保 livox_ros_driver2 使用的是 `msg_MID360.launch`，因为 Point-LIO 需要带点时间戳的 Livox 自定义点云消息。

## Point-LIO 到 MAVROS vision_pose 桥接 {#point-lio-mavros-vision-pose-bridge}

进入 [EKF Gate 与融合检查](../flight-debug/ekf-gate.md) 前，必须先把 Point-LIO 输出位姿转换并发布到：

```text
/mavros/vision_pose/pose
```

桥接节点需要满足：

- 订阅 Point-LIO 输出的位姿 / 里程计话题。
- 使用 ROS 当前时间戳或可靠的传感器时间戳。
- 输出 `geometry_msgs/PoseStamped`。
- 坐标系按 MAVROS `vision_pose` 的 ENU 约定处理。
- yaw 方向要与 PX4 `vision yaw fusion` 一致。

桥接完成后用以下命令检查：

```bash
rostopic hz /mavros/vision_pose/pose
rostopic echo /mavros/vision_pose/pose
```

如果 `/mavros/vision_pose/pose` 没有稳定输出，不要继续做 EKF 融合检查；先回到 Point-LIO 输出、桥接节点和 MAVROS 话题排查。

## 调参注意

!!! warning "调参仅作排查参考"
    调参内容仅供雷达定位输出异常、漂移或初始化问题时排查参考，正常安装验证时不需要逐项改参数。

### mapping_avia.launch

重点检查：

```yaml
space_down_sample: 1
filter_size_surf: 0.25
```

- `space_down_sample`：是否开启降采样。
- `filter_size_surf`：体素滤波的体素边长。

### avia.yaml

预处理盲区：

```yaml
preprocess:
  blind: 0.5
```

`blind` 表示盲区距离，这个距离内的点会被直接丢弃。

初始化时防止里程计跳变相关参数：

```yaml
initialization:
  init_map_size: 1000
  init_accum_by_frame_en: false
  init_accum_frame_num: 10
  origin_lock_en: false
  origin_lock_frame_num: 10
  origin_lock_threshold: 0.05
```

含义：

- `init_map_size`：原逻辑下用于地图初始化累积的点数阈值；仅在 `init_accum_by_frame_en=false` 时生效。
- `init_accum_by_frame_en`：`false` 表示按点数累积建图，`true` 表示按帧数在原点累积建图。
- `init_accum_frame_num`：按帧数初始化时累积的雷达帧数；仅在 `init_accum_by_frame_en=true` 时生效。
- `origin_lock_en`：初始化后若位姿偏离原点超过阈值，是否强制回到原点。
- `origin_lock_frame_num`：初始化后开启原点锁定检测的帧数。
- `origin_lock_threshold`：触发原点锁定的欧式距离阈值，单位 m。

## 关键检查项

- LiDAR 和 IMU 必须同步。
- 按实际 IMU 设置 `satu_acc`、`satu_gyro` 和 `acc_norm`。
- 如果外参已经给定，建议设置 `extrinsic_est_en=false`。
- `lid_topic` 和 `imu_topic` 必须与当前 livox_ros_driver2 / 飞控 IMU 话题一致。
- `extrinsic_T` 和 `extrinsic_R` 表示 LiDAR 在 IMU body frame 下的位置和旋转，必须按实机安装关系确认。
