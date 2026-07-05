# EKF Gate 与融合检查

本页用于确认 Point-LIO 经过 MAVROS `vision_pose` 入口后，是否已经进入 PX4 EKF2 融合。使用本页前，应先完成 NUC 侧 MAVROS、Mid-360、Point-LIO 和 `/mavros/vision_pose/pose` 桥接。

!!! warning "先完成 NUC 配置"
    EKF Gate 与融合检查必须在完成 NUC 配置后再进行。不要在 Mid-360、MAVROS、Point-LIO 和 `vision_pose` 桥接未完成时进入本页检查。

!!! warning "先有 vision_pose，再看 EKF"
    如果 `/mavros/vision_pose/pose` 没有稳定输出，不要继续调 EKF Gate。先回到 NUC 启动链路、Point-LIO 输出和桥接节点排查。

## 操作检查顺序

### 1. 检查 MAVROS 连接

NUC 启动 MAVROS 后，先确认飞控连接：

```bash
rostopic echo /mavros/state
```

应看到：

```text
connected: True
```

### 2. 检查飞控 IMU 数据

```bash
rostopic echo /mavros/imu/data
```

有连续数据且随飞控姿态变化，说明 MAVROS 能收到飞控基础数据。

### 3. 检查 vision_pose 输入

启动 Mid-360、Point-LIO 和 `vision_pose` 桥接后，检查外部定位是否发布：

```bash
rostopic hz /mavros/vision_pose/pose
rostopic echo /mavros/vision_pose/pose
```

### 4. 检查 PX4 本地位置输出

再检查本地位置输出是否稳定：

```bash
rostopic hz /mavros/local_position/odom
rostopic echo /mavros/local_position/odom
```

## 通过标准

- `/mavros/state` 显示 `connected: True`。
- `/mavros/imu/data` 有连续输出。
- `/mavros/vision_pose/pose` 有稳定频率。
- `/mavros/local_position/odom` 在外部定位启动后稳定输出，并确认话题格式对齐。
- 移动或轻微晃动机体时，本地位置不会明显跳变或持续漂移。

## 参数说明

### 当前固定参数

| 参数 | 固定值 | 作用 |
| --- | --- | --- |
| `EKF2_AID_MASK` | `24` | 融合 vision position 和 vision yaw。 |
| `EKF2_EVP_GATE` | `15.0 SD` | 视觉位置 innovation gate。 |
| `EKF2_HDG_GATE` | `6.0 SD` | 外部 yaw / heading innovation gate。 |
| `EKF2_EVP_NOISE` | `0.01 m` | 位置观测噪声。 |
| `EKF2_EVA_NOISE` | `2.86 deg` | 姿态观测噪声。 |
| `EKF2_EV_DELAY` | `5.0 ms` | 外部视觉延迟初值。 |
| `EKF2_EV_POS_Z` | `-0.080 m` | 外部定位传感器 Z 方向安装偏移。 |

### SD 的含义

`SD` 是 Standard Deviation，表示 innovation 残差标准差的倍数，不是某个传感器原始数据单位。

EKF 融合观测时会计算：

```text
innovation = EKF预测值 - 视觉观测值
innovation_variance = EKF预测不确定性 + 视觉观测不确定性
```

Gate 判断等价于：

```text
|innovation| <= gate * sqrt(innovation_variance)
```

所以：

- `EKF2_EVP_GATE = 15` 表示视觉位置残差在 15 倍位置 innovation 标准差以内时可被接受。
- `EKF2_HDG_GATE = 6` 表示 yaw / heading 残差在 6 倍 heading innovation 标准差以内时可被接受。
- `SD` 本身无量纲，但实际门限有物理量纲：位置 gate 最终对应米，heading gate 最终对应弧度。

### 为什么调大 Gate 会缓解漂移

当前记录中，将 `EKF2_EVP_GATE` 从 `5 SD` 调为 `15 SD`，`EKF2_HDG_GATE` 从 `2.6 SD` 调为 `6 SD` 后，漂移明显缓解。

这通常说明大幅摇晃或运动时，视觉 / Point-LIO 的 innovation 可能超过原来的 gate，被 EKF 短时拒绝。调大 gate 后，观测更容易持续进入融合。

!!! warning "Gate 变大不是根本修复"
    大 gate 更宽松，也更容易吃进异常视觉数据。它适合先验证问题，但长期仍要继续检查时间戳、坐标系、yaw 方向、`EKF2_EV_POS` 杆臂和视觉 covariance 是否低估。

## 排查方向

如果 EKF 融合不稳定或漂移，按链路顺序排查：

1. MAVROS 连接：检查 `/mavros/state` 是否稳定保持 `connected: True`。
2. Mid-360 / Point-LIO：检查雷达数据和 Point-LIO 输出是否跳变。
3. `vision_pose` 坐标系：检查是否满足 MAVROS `vision_pose` 的 ENU 约定。
4. yaw 方向：检查 yaw 是否方向相反或存在 90° / 180° 偏差。
5. 时间戳：检查是否用当前 ROS time，延迟是否接近 `EKF2_EV_DELAY`。
6. 杆臂偏移：检查 Mid-360 相对飞控的安装偏移，尤其是 `EKF2_EV_POS_Z`。
7. covariance：检查是否把过小的 covariance 送入 MAVROS，导致 EKF 过度信任异常观测。
