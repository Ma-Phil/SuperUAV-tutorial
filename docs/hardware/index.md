# 一、硬件连线总览

本部分先把整机硬件连接关系讲清楚。当前连线分成三类：

- 动力供电：6S 电池 / XT60、电调与外接 50V 2200uF 电容、电机。
- 控制信号：F60 MINI 4IN1 V2 电调 8pin 口—— PIX6C MINI 的 FMU PWM OUT，飞控——遥控接收器。
- 飞控与外设连接：5V 降压电路为飞控供电，19V 降压电路为雷达 / NUC 供电（同时含雷达——NUC的网口线材）。

!!! note "机架与电机参考"
    本教程中的机架结构与电机选型参考 [hku-mars/SUPER-Hardware](https://github.com/hku-mars/SUPER-Hardware)，实际安装时以当前 Super 飞机实物为准。

!!! warning "安全边界"
    焊接和通电前，需核对实物丝印、接口方向、正负极。碳板导电，任何裸露的电路板不要直接接触碳板；首次电机测试不要安装桨叶。

<iframe class="diagram-frame hardware-frame" src="../assets/diagrams/hardware-wiring.html" title="硬件连线总览"></iframe>

## 本部分操作顺序

<div class="section-grid" markdown>

<div class="section-card" markdown>
### 1. 动力与供电
XT60 主电源、电调与外接电容、电机、19V 降压、5V 降压与飞控供电。  
[查看](power.md)
</div>

<div class="section-card" markdown>
### 2. 电调到飞控信号
F60 电调 8pin SH1.0 转杜邦，接 PIX6C MINI FMU PWM OUT / AUX OUT。  
[查看](esc-fc-signal.md)
</div>

<div class="section-card" markdown>
### 3. 飞控到遥控接收器
根据现有示意图连接飞控与遥控接收器。  
[查看](receiver.md)
</div>

<div class="section-card" markdown>
### 4. 通电前检查与软件验证入口
按检查清单逐项确认，再进入 QGC 参数和电机测试。  
[查看](checklist.md)
</div>

</div>
