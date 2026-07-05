# 二、飞控调试

本部分是硬件安装完成后的第一轮软件验证，面向 **PX4 v1.13.3 + V6C / PIX6C 类飞控 + 4in1 电调 + 倒置电机四旋翼**。目标不是直接起飞，而是先确认飞控参数、输出协议、电机编号和转向都与实际接线一致。

调试前必须先完成 [通电前检查](../hardware/checklist.md)。调试电脑可以是普通笔记本，也可以是飞机机载 NUC；只要已安装 QGroundControl 并能通过 USB 或串口稳定连接飞控即可。

!!! warning "调试安全边界"
    导入参数、校准、MAVLink Shell 和电机测试前必须卸桨。电机倒置安装时，不要从机体下方仰视判断转向，应统一按机体上方俯视方向对照标准图。

## 本部分操作顺序

1. 用 QGC 连接飞控，确认固件版本为 PX4 v1.13.3，保存一份当前参数备份。
2. 根据当前情况选择参数配置方式：
   - 已有同型号成熟参数文件时，使用 [方案一：导入已有参数文件](import-params.md)。
   - 新飞控、参数文件不确定或需要逐项确认时，使用 [方案二：新飞控手动调参](manual-params.md)。
3. 设置飞控安装方向、机架方向，并完成陀螺仪、加速度计、水平姿态等传感器校准。
4. 检查 `DSHOT_CONFIG = DShot600` 和 `SYS_USE_IO = Disable PWM`，确认电调输出协议与接线方式一致。
5. 在 [电机测试与反转](motor-test.md) 中逐个测试 Motor 1~4，核对电机编号和转向。编号不对先改 ESC1~ESC4 信号线；编号正确但方向错误，优先用 DShot 反转；最后才交换电机三相线。
6. 电机编号与转向全部通过后，进入 [手飞测试流程](manual-flight-test.md)。
7. 手飞测试通过后，先完成 [NUC 配置](../nuc/index.md)，启动 MAVROS、Mid-360、Point-LIO 和 `vision_pose` 桥接。
8. 完成 NUC 配置后，再在 [EKF Gate 与融合检查](ekf-gate.md) 中确认外部定位已进入 PX4 EKF2。
9. EKF Gate 与融合检查通过后，再进入 [定点环绕自主飞行测试流程](orbit-autonomous-test.md)。

## 进入 NUC 前的通过标准

- QGC 能稳定连接飞控，参数已备份。
- PX4 固件版本确认为 v1.13.3。
- 飞控朝向和姿态校准完成，HUD 姿态与实物一致。
- `DSHOT_CONFIG = DShot600`，`SYS_USE_IO = Disable PWM`。
- Motor 1~4 的物理位置和转向全部符合 [电机测试与反转](motor-test.md) 中的标准。
- 全部电机测试都在 **不安装桨叶** 的状态下完成。
- [手飞测试流程](manual-flight-test.md) 已完成。

## 参考

- [QGroundControl 参数导入/导出](https://docs.qgroundcontrol.com/master/en/qgc-user-guide/setup_view/parameters.html)
- [QGroundControl 传感器与飞控朝向设置](https://docs.qgroundcontrol.com/master/en/qgc-user-guide/setup_view/sensors_px4.html)
- [QGroundControl 电机测试](https://docs.qgroundcontrol.com/master/en/qgc-user-guide/setup_view/motors.html)
- [PX4 v1.13 外部视觉/运动捕捉定位融合](https://docs.px4.io/v1.13/en/ros/external_position_estimation)
