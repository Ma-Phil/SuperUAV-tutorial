# 方案二：新飞控手动调参

本方案用于新飞控、参数文件不可用或需要逐项核对的情况。目标配置固定为：**PX4 v1.13.3 + PX6C / V6C 类飞控 + 四旋翼 + 4in1 电调 + DShot600 + Mid-360 / Point-LIO / MAVROS 外部视觉定位融合**。

!!! warning "每轮修改后重启并复查"
    涉及 DShot、`SYS_USE_IO`、断路器和 EKF2 的参数修改后，建议重启飞控，再重新下载参数确认已经保存。

## 操作步骤

1. 用 QGC 连接飞控，确认固件为 **PX4 v1.13.3**。
2. 在 Airframe 中选择四旋翼机架。
3. 设置飞控安装方向为 **Pitch 180°**。
4. 完成 Gyroscope、Accelerometer、Level Horizon 校准。
5. 完成遥控器校准和飞行模式检查。
6. 按下面的参数说明设置电机输出、断路器、通信和 EKF 外部定位参数。
7. 保存参数并重启飞控，再回到 QGC 下载参数确认保存成功。
8. 执行 [电机测试与反转](motor-test.md)，再进入 NUC 和 EKF 检查。

## 参数说明

### 电机与输出参数

| 参数 | 固定值 | 说明 |
| --- | --- | --- |
| `DSHOT_CONFIG` | `DShot600` | 使用 DShot600 控制电调。 |
| `SYS_USE_IO` | `Disable PWM` | 当前电调接 FMU PWM OUT / AUX OUT，DShot 应走 FMU 输出。 |
| `MC_AT_EN` | `Disable` | 关闭自动调参。 |
| `MAN_ARM_GESTURE` | `Disable` | 关闭油门杆手势解锁。 |

如果电机没有响应，不要先改成 PWM；先回到 [电机测试与反转](motor-test.md) 检查信号线、输出口、QGC 电机测试和 MAVLink Console。

### EKF 外部定位参数

Point-LIO 在 NUC 上输出定位，经过 MAVROS 的 `vision_pose` 入口送给 PX4 EKF2。当前固定融合位置和 yaw。

这里先完成参数设置，不代表已经完成 EKF 融合验证。实际融合要在 NUC 侧 MAVROS、Mid-360、Point-LIO 和 `vision_pose` 桥接启动后，再进入 [EKF Gate 与融合检查](ekf-gate.md)。

| 参数 | 固定值 | 说明 |
| --- | --- | --- |
| `EKF2_AID_MASK` | `24` | 只勾选 `vision position fusion` 和 `vision yaw fusion`。 |
| `EKF2_HGT_MODE` | `Vision` | Z 高度来源使用外部视觉 / 里程计。 |
| `EKF2_EVP_GATE` | `15.0 SD` | 视觉位置 innovation gate。当前 Super 机调大后漂移明显缓解。 |
| `EKF2_HDG_GATE` | `6.0 SD` | 外部 yaw / heading innovation gate。 |
| `EKF2_EVP_NOISE` | `0.01 m` | 外部位置观测噪声。 |
| `EKF2_EVA_NOISE` | `2.86 deg` | 外部姿态观测噪声。 |
| `EKF2_EV_DELAY` | `5.0 ms` | 外部视觉相对 IMU 的延迟初值。 |
| `EKF2_EV_POS_Z` | `-0.080 m` | 外部定位传感器相对飞控 / 机体系的 Z 方向偏移。 |

`EKF2_EV_POS_X` 和 `EKF2_EV_POS_Y` 若没有实测值，先保持参数文件中的固定值或默认值；最终应按 Mid-360 相对飞控的安装位置实测。

### 断路器与通信参数

| 参数 | 固定值 | 说明 |
| --- | --- | --- |
| `CBRK_SUPPLY_CHK` | `894281` | 关闭电池检测。仅在供电方案已单独确认时使用。 |
| `CBRK_USB_CHK` | `197848` | 允许 USB 连接时启动相关流程。调试时必须卸桨。 |
| `CBRK_IO_SAFETY` | `22027` | 关闭安全开关检查。 |
| `SER_TEL1_BAUD` | `921600 8N1` | 若后续改用 TELEM1 串口，此值需与 NUC 侧一致；当前 USB MAVROS 主路径使用 `/dev/ttyACM0:921600`。 |

## 调参后的检查顺序

1. 保存参数并重启飞控。
2. 进入 QGC HUD，确认飞控姿态与实物一致。
3. 执行 [电机测试与反转](motor-test.md)。
4. 启动 NUC 侧 MAVROS，确认 `/mavros/state` 中 `connected: True`。
5. 启动 Mid-360、Point-LIO 和 `vision_pose` 桥接后，进入 [EKF Gate 与融合检查](ekf-gate.md)。

## 参考

- [PX4 v1.13 外部视觉/运动捕捉定位融合](https://docs.px4.io/v1.13/en/ros/external_position_estimation)
- [QGroundControl 参数页](https://docs.qgroundcontrol.com/master/en/qgc-user-guide/setup_view/parameters.html)
- [QGroundControl 电机测试](https://docs.qgroundcontrol.com/master/en/qgc-user-guide/setup_view/motors.html)
- [QGroundControl MAVLink Console](https://docs.qgroundcontrol.com/master/en/qgc-user-guide/analyze_view/mavlink_console.html)
