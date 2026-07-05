# PX6C USB 连接 MAVROS

本机固定使用 USB 线连接 NUC 与 PX6C / V6C 类飞控。Pixhawk 6C 通过 USB 连接时通常显示为 `/dev/ttyACM0`。

## 安装 MAVROS

本教程主路径使用 MAVROS `master` 分支源码工作空间。NUC 总览中的固定源码仓库表已经把 MAVROS 列为必装项，本页用于确认 MAVROS 工作空间、USB 权限、`fcu_url` 和连接验证。

### 主路径：源码工作空间

启动脚本默认从以下工作空间加载 MAVROS，源码固定放在 `src/mavros`：

```text
~/rosprojects/init_mav/mavros_ws/src/mavros
```

安装：

```bash
mkdir -p ~/rosprojects/init_mav/mavros_ws/src
cd ~/rosprojects/init_mav/mavros_ws/src
git clone https://github.com/mavlink/mavros.git mavros

cd ~/rosprojects/init_mav/mavros_ws
source /opt/ros/noetic/setup.bash
rosdep install --from-paths src --ignore-src -r -y
catkin build
source devel/setup.bash
```

如果实际源码工作空间不同，需要同步修改 [启动脚本与启动顺序](startup.md) 中的 `source` 路径。

### 备选：二进制快速验证

二进制包只用于快速验证 MAVROS 是否能连接飞控，不作为本教程最终启动脚本的主路径：

```bash
sudo apt update
sudo apt install -y ros-noetic-mavros ros-noetic-mavros-extras ros-noetic-mavros-msgs
```

无论使用源码还是二进制，都需要安装 GeographicLib 地理数据：

```bash
wget https://raw.githubusercontent.com/mavlink/mavros/master/mavros/scripts/install_geographiclib_datasets.sh
sudo bash ./install_geographiclib_datasets.sh
```

## 配置 USB 权限

把当前用户加入 `dialout` 组：

```bash
sudo usermod -a -G dialout $USER
```

执行后重启 NUC，或注销后重新登录。

## 确认飞控设备名

```bash
ls /dev/ttyACM* /dev/ttyUSB* 2>/dev/null
dmesg | grep -i "ttyACM\|ttyUSB\|pixhawk"
ls -la /dev/ttyACM* /dev/ttyUSB* 2>/dev/null
```

成功识别时，PX6C USB 通常是：

```text
/dev/ttyACM0
```

## 固定 MAVROS fcu_url

本教程固定使用：

```bash
roslaunch mavros px4.launch fcu_url:=/dev/ttyACM0:921600
```

如果直接修改 `px4.launch`，对应参数为：

```xml
<arg name="fcu_url" default="/dev/ttyACM0:921600" />
```

补充记录中提到 `57600` 可用，但本教程为了与启动脚本、参数表保持一致，主路径固定为 `921600`。

## 验证 MAVROS 连接

启动：

```bash
roslaunch mavros px4.launch fcu_url:=/dev/ttyACM0:921600
```

检查状态：

```bash
rostopic echo /mavros/state
rostopic hz /mavros/state
```

成功标志：

```text
connected: True
```

示例：

```text
connected: True
armed: False
guided: False
manual_input: True
mode: "MANUAL"
system_status: 3
```

检查 IMU 数据：

```bash
rostopic echo /mavros/imu/data
```

有连续数据且数值变化正常，即说明 NUC 与飞控的 MAVROS 基础连接可用。

## 常见问题

- 如果 `/dev/ttyACM0` 不存在，重新插拔 USB 并检查 `dmesg`。
- 如果权限不足，确认已经加入 `dialout` 并重启。
- 如果 QGC 正在占用同一个 USB 连接，MAVROS 可能无法稳定连接。调试时避免 QGC 和 MAVROS 同时抢占同一飞控串口。
