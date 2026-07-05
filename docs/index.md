# 无人机制作教程

<div class="tutorial-hero">
本教程面向固定 Super 无人机配置：PX6C / V6C 类飞控、PX4 v1.13.3、F60 4in1 电调、Livox Mid-360、NUC Ubuntu 20.04 + ROS Noetic、Point-LIO-new 与 MAVROS 外部定位融合。
</div>

## 教程使用方式 {#how-to-use}

推荐按照下方安装配置依赖进度图顺序进行操作。

!!! warning "安全边界"
    硬件接线、QGC 参数设置、MAVLink Console 和电机测试阶段都必须卸桨。电调和电机接好后，不要凭硬件接线判断最终转向，必须用 QGC 点动 Motor 1~4 逐个确认。

## 安装配置依赖进度图

<div class="progress-map" markdown>

<div class="progress-phase-row">
<div class="progress-phase-corner">泳道 / 阶段</div>
<div class="progress-phase-cell">0 准备</div>
<div class="progress-phase-cell">1 硬件安装</div>
<div class="progress-phase-cell">2 硬件检查</div>
<div class="progress-phase-cell">3 飞控配置</div>
<div class="progress-phase-cell">4 电机门禁</div>
<div class="progress-phase-cell">5 NUC 安装</div>
<div class="progress-phase-cell">6 外部定位</div>
<div class="progress-phase-cell">7 EKF 检查</div>
</div>

<div class="progress-lane" markdown>
<div class="progress-lane-title">硬件装配</div>
<a class="progress-step progress-step-hardware progress-col-1 progress-span-1" href="hardware/power.html">
<strong>动力与供电</strong>
<span>先测电压</span>
</a>
<a class="progress-step progress-step-gate progress-col-2 progress-span-1" href="hardware/checklist.html">
<strong>通电前检查</strong>
<span>电源/信号/机械</span>
</a>
</div>

<div class="progress-lane" markdown>
<div class="progress-lane-title">控制链路</div>
<div class="progress-branch progress-col-1 progress-span-1">
<a class="progress-step progress-step-hardware" href="hardware/esc-fc-signal.html">
<strong>电调信号</strong>
<span>ESC1~4/GND</span>
</a>
<a class="progress-step progress-step-hardware" href="hardware/receiver.html">
<strong>遥控接收器</strong>
<span>供电并共地</span>
</a>
</div>
<a class="progress-step progress-step-gate progress-col-2 progress-span-1" href="hardware/checklist.html">
<strong>信号线核对</strong>
<span>不接电调V</span>
</a>
<div class="progress-branch progress-col-3 progress-span-1">
<a class="progress-step progress-step-flight" href="flight-debug/import-params.html">
<strong>QGC 参数</strong>
<span>导入或手调</span>
</a>
<a class="progress-step progress-step-flight" href="flight-debug/manual-params.html">
<strong>DShot / SYS_USE_IO</strong>
<span>走FMU输出</span>
</a>
</div>
<a class="progress-step progress-step-gate progress-col-4 progress-span-1" href="flight-debug/motor-test.html">
<strong>Motor 1~4 门禁</strong>
<span>编号和转向</span>
</a>
</div>

<div class="progress-lane" markdown>
<div class="progress-lane-title">NUC 主链路</div>
<a class="progress-step progress-step-gate progress-col-4 progress-span-1" href="flight-debug/motor-test.html">
<strong>通过后进入</strong>
<span>先过电机门禁</span>
</a>
<a class="progress-step progress-step-nuc progress-col-5 progress-span-1" href="nuc/index.html">
<strong>NUC 依赖 / 源码</strong>
<span>init_mav路径</span>
</a>
<div class="progress-branch progress-col-6 progress-span-1">
<a class="progress-step progress-step-nuc" href="nuc/mid360.html">
<strong>Mid-360 配置</strong>
<span>IP/config</span>
</a>
<a class="progress-step progress-step-nuc" href="nuc/px6c-usb-mavros.html">
<strong>MAVROS 连接</strong>
<span>921600</span>
</a>
<a class="progress-step progress-step-nuc" href="nuc/point-lio.html#point-lio-mavros-vision-pose-bridge">
<strong>Point-LIO / vision_pose</strong>
<span>稳定输出</span>
</a>
</div>
<a class="progress-step progress-step-gate progress-col-7 progress-span-1" href="flight-debug/ekf-gate.html">
<strong>EKF 融合检查</strong>
<span>看 odom 输出</span>
</a>
</div>

<div class="progress-lane" markdown>
<div class="progress-lane-title">可选分支</div>
<a class="progress-step progress-step-optional progress-col-5 progress-span-1" href="nuc/realsense.html">
<strong>可选：RealSense</strong>
<span>不阻塞主链路</span>
</a>
<a class="progress-step progress-step-optional progress-col-6 progress-span-1" href="nuc/realsense.html">
<strong>相机验证可选</strong>
<span>需要时再启动</span>
</a>
<a class="progress-step progress-step-optional progress-col-7 progress-span-1" href="nuc/realsense.html">
<strong>不阻塞主链路</strong>
<span>Mid-360优先</span>
</a>
</div>

</div>

**关键依赖判断：**

- 电机测试通过后，安装桨叶进行手飞测试，手飞测试通过后再进行 NUC / EKF 调试。
- `/mavros/vision_pose/pose` 需要有稳定输出，然后再监控 `/mavros/local_position/odom` 话题格式对齐。
- RealSense 是可选外设，不作为主定位链路的前置条件。

## 推荐操作路径

1. 先看 [硬件连线总览](hardware/index.md)，确认动力供电、控制信号和外设连接三类链路。
2. 完成 [动力与供电](hardware/power.md)、[电调到飞控信号](hardware/esc-fc-signal.md) 和 [飞控到遥控接收器](hardware/receiver.md)。
3. 按 [通电前检查](hardware/checklist.md) 逐项确认电源、信号、机械固定和卸桨状态。
4. 进入 [飞控调试](flight-debug/index.md)，用 QGC 备份参数，选择导入已有参数或按新飞控流程手动设置。
5. 完成 [电机测试与反转](flight-debug/motor-test.md)，确认 Motor 1~4 的物理位置和转向。
6. 配置 [NUC 软件链路](nuc/index.md)，检查 MAVROS、Mid-360、Point-LIO-new 和 `vision_pose` 输出。
7. 最后做 [EKF Gate 与融合检查](flight-debug/ekf-gate.md)，确认外部定位已进入 PX4 EKF2。

## 专题入口

<div class="section-grid" markdown>

<div class="section-card section-ready" markdown>
### 一、硬件连线
动力供电、电调到电机、飞控供电、电调到飞控信号线、飞控到遥控接收器和通电前检查。  
[进入硬件连线](hardware/index.md)
</div>

<div class="section-card section-ready" markdown>
### 二、飞控调试
QGC 连接飞控、参数导入或手动设置、飞控朝向与传感器校准、电机编号和转向测试。  
[进入飞控调试](flight-debug/index.md)
</div>

<div class="section-card section-ready" markdown>
### 三、NUC 配置
Ubuntu 20.04 + ROS Noetic 下配置 Mid-360、Livox-SDK2 / livox_ros_driver2、Point-LIO-new、MAVROS USB 连接和可选 RealSense。  
[进入 NUC 配置](nuc/index.md)
</div>

</div>
