# 通电前检查

完成硬件连线后，按这页逐项检查。任何一项不确定，都先不要上电。

## 电源检查

- XT60 正负极确认无误。
- 电调电源输入焊点没有锡桥。
- 50V 2200uF 电容极性正确。
- 19V 降压输出确认后，再接雷达 / NUC 支路。
- 5V 降压输出确认后，再接飞控 Power 接口。

## 信号检查

- 电调 8pin 线只接 ESC1、ESC2、ESC3、ESC4、GND 五根。
- T、C、V 三根线已单独绝缘。
- 电调信号接 FMU PWM OUT / AUX OUT，不接 MAIN OUT。
- 电调 GND 接到 FMU PWM OUT 的 `- / GND`。
- 电调 V 不接 FMU PWM OUT 的 `+ / VDD_SERVO`。

## 机械与安全检查

- 电机螺丝长度合适，没有顶到线圈。
- 电机线、供电线、信号线固定，不会被桨叶扫到。
- 第一次上电和电机测试不安装桨叶。
- 电池、飞控、NUC、雷达线束固定，避免飞行中拉扯接口。

## 进入硬件后软件验证

硬件检查通过后，不要直接进入 NUC 或 EKF 调试。先用 QGC 做第一轮软件验证，确认飞控参数、输出协议和电机实际响应都与硬件接线一致。

!!! warning "验证前仍然不装桨"
    QGC 连接、参数导入、MAVLink Console 和电机测试都必须在卸桨状态下完成。只要还没有完成 Motor 1~4 编号与转向测试，就不要安装桨叶。

必须完成：

- 用 QGC 连接飞控，确认固件版本为 **PX4 v1.13.3**，并先备份当前参数。
- 选择 [导入已有参数文件](../flight-debug/import-params.md) 或 [新飞控手动调参](../flight-debug/manual-params.md)。
- 检查 `DSHOT_CONFIG = DShot600`。
- 检查 `SYS_USE_IO = Disable PWM`，确认当前电调输出走 FMU PWM OUT / AUX OUT。
- 进入 [电机测试与反转](../flight-debug/motor-test.md)，用 QGC 逐个点动 Motor 1~4。
- 如果 Motor ID 控制的物理位置不对，先调整 ESC1~ESC4 信号线顺序。
- 如果 Motor ID 正确但转向错误，优先执行 `dshot reverse -m ID` 和 `dshot save -m ID`。
- 如果 DShot 反转无效，断电后再交换对应电机任意两根三相线。

通过标准：

- QGC Motor 1 控制前右电机，方向 CCW。
- QGC Motor 2 控制后左电机，方向 CCW。
- QGC Motor 3 控制前左电机，方向 CW。
- QGC Motor 4 控制后右电机，方向 CW。
- 每次点动测试前都确认桨叶未安装。

以上全部通过后，再进入 [NUC 配置](../nuc/index.md) 和 [EKF Gate 与融合检查](../flight-debug/ekf-gate.md)。
