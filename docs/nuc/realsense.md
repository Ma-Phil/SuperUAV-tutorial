# RealSense 可选配置

RealSense 是本教程中的可选外设，不作为 Mid-360 + Point-LIO 主定位链路的必要条件。当前记录采用源码安装：

```text
librealsense v2.50.0
realsense-ros 2.3.2
```

固定路径：

```text
~/rosprojects/init_mav/realsense_SDK
~/rosprojects/init_mav/realsense_ros_ws/src/realsense-ros
```

## 安装 librealsense SDK

```bash
mkdir -p ~/rosprojects/init_mav
cd ~/rosprojects/init_mav
git clone -b v2.50.0 https://github.com/IntelRealSense/librealsense.git realsense_SDK
cd realsense_SDK

sudo apt-get install -y libudev-dev pkg-config libgtk-3-dev libusb-1.0-0-dev libglfw3-dev libssl-dev cmake build-essential

./scripts/setup_udev_rules.sh

# Ubuntu 20.04/22.04 LTS 旧内核
./scripts/patch-realsense-ubuntu-lts.sh

# 如果是 HWE 新内核，可改用：
# ./scripts/patch-realsense-ubuntu-lts-hwe.sh

mkdir build
cd build
cmake ../ -DBUILD_EXAMPLES=true -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)
sudo make install
```

验证：

```bash
realsense-viewer
rs-enumerate-devices
```

`realsense-viewer` 能打开图形界面，`rs-enumerate-devices` 能看到设备信息，即 SDK 安装基本可用。

## 安装 realsense-ros1

```bash
mkdir -p ~/rosprojects/init_mav/realsense_ros_ws/src
cd ~/rosprojects/init_mav/realsense_ros_ws/src

git clone -b 2.3.2 https://github.com/IntelRealSense/realsense-ros.git realsense-ros

cd ~/rosprojects/init_mav/realsense_ros_ws
rosdep install --from-paths src --ignore-src -r -y
catkin_make
source devel/setup.bash
```

启动：

```bash
roslaunch realsense2_camera rs_camera.launch
```

## 常见问题

- 如果相机无法识别，先运行 `rs-enumerate-devices` 判断 SDK 是否能看到设备。
- 如果 USB 带宽不足，换 NUC 的 USB3 口并减少图像分辨率 / 帧率。
- 如果内核补丁失败，确认当前内核版本，再选择 `patch-realsense-ubuntu-lts.sh` 或 `patch-realsense-ubuntu-lts-hwe.sh`。
- 如果只需要 Mid-360 + Point-LIO 定位，可以暂时不启动 RealSense，减少调试变量。
