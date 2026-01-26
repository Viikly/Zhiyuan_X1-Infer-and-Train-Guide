# 智元灵犀X1模型推理与训练指南

## 概述

智元灵犀X1是一款模块化、高自由度人形机器人。其软件栈分为两部分：

1. **推理软件 (`infer`)**：基于AimRT中间件，包含模型推理、仿真、平台驱动和手柄控制模块，用于在仿真或真实机器人上运行训练好的强化学习模型。(本指南聚焦于模拟运行）
2. **训练代码 (`train`)**：基于Isaac Lab/Isaac Gym，用于训练强化学习运动控制策略。

---
## 目录
- [1. 推理软件（Infer）安装与运行](#推理软件（Infer）安装与运行)
- [1.1 系统要求与依赖安装](#系统要求与依赖安装)
- [1.1.1 基础编译环境](#基础编译环境)
- [1.1.2 安装ROS2 Humble](#安装ROS2-Humble)
- [1.1.3 安装仿真环境依赖](#安装仿真环境依赖)
- [1.2 获取代码与编译](#获取代码与编译)
- [1.3 运行仿真](#运行仿真)
- [1.4 手柄操控](#手柄操控)
- [2. 模型训练（Train）安装与运行](#模型训练（Train）安装与运行)
- [2.1 环境配置](#环境配置)
- [2.2 训练与评估](#训练与评估)
- [3 综合工作流程](#综合工作流程)
- [4.常见问题](#常见问题)
- [5.许可](#许可)

## 1.推理软件 (`infer`) 安装与运行
<a id="推理软件（Infer）安装与运行"></a>

### 1.1 系统要求与依赖安装
<a id="系统要求与依赖安装"></a>

### 推荐配置
| 组件 | 推荐配置 |
| :--- | :--- |
| **操作系统** | Ubuntu 22.04 LTS |
| **CPU** | Intel Core i7 (第7代) / AMD Ryzen 5 或更高 |
| **内存** | 32 GB |
| **GPU** | NVIDIA GeForce RTX 3070 (8GB显存) |
| **GPU驱动** | 版本 535.129.03+ |
| **存储** | 50 GB 可用空间的 SSD |

#### 1.1.1 基础编译环境
<a id="基础编译环境"></a>
- 安装[GCC-13](https://www.gnu.org/software/gcc/gcc-13/)。
- 安装[cmake](https://cmake.org/download/)3.26以上版本。
- 安装[ONNX Runtime](https://github.com/microsoft/onnxruntime)。
```bash
sudo apt update
sudo apt install -y build-essential cmake git libprotobuf-dev protobuf-compiler

git clone --recursive https://github.com/microsoft/onnxruntime

cd onnxruntime
./build.sh --config Release --build_shared_lib --parallel

cd build/Linux/Release/
sudo make install
```

#### 1.1.2 安装ROS2 Humble
<a id="安装ROS2-Humble"></a>

1.  **确保支持`UTF-8`**：
    ```bash
    sudo apt update && sudo apt install locales
    sudo locale-gen en_US en_US.UTF-8
    sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
    export LANG=en_US.UTF-8
    ```
2.  **添加ROS2 apt仓库**：
    确保`Ubuntu Universe repository`生效
    ```bash
    sudo apt install software-properties-common
    sudo add-apt-repository universe
    ```
    添加ROS2的github仓库。**Github仓库需要较稳定的网络环境，如后续安装失败可自行寻找Gitee镜像替换指令中的`Github`地址**。
    ```bash
    sudo apt update && sudo apt install curl -y
    export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F\" '{print $4}')
    curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo     ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
    sudo dpkg -i /tmp/ros2-apt-source.deb
    ```
3.  **安装ROS2包**:
    ```bash
    sudo apt update
    sudo apt upgrade
    ```
    
    安装`ROS2 Humbel Desktop`版本包
    ```bash
    sudo apt install ros-humble-desktop
    ```
    
#### 1.1.3 安装仿真环境依赖
<a id="安装仿真环境依赖"></a>

```bash
sudo apt install jstest-gtk libglfw3-dev libdart-external-lodepng-dev
```

### 1.2 获取代码与编译
<a id="获取代码与编译"></a>

由于`AimRT`依赖较多，建议使用`Gitee`镜像加速下载。**若在使用仓库提供的`Gitee`源环境变量仍出现网络不通情况可自行在`Gitee`上寻找可用的`AimRT`镜像手动安装**。
```bash
# 获取仓库代码
git clone https://github.com/AgibotTech/agibot_x1_infer.git

# 进入仓库
cd agibot_x1_infer/

# 使用提供的 gitee 源环境变量
source url_gitee.bashrc

# 确保 ROS2 环境已激活
source /opt/ros/humble/setup.bash

# 编译
./build.sh $DOWNLOAD_FLAGS
# 运行测试 (可选)
./test.sh $DOWNLOAD_FLAGS
```
编译完成后，可执行文件位于`agibot_x1_infer/build/`目录下。

### 1.3 运行仿真
<a id="运行仿真"></a>

**在运行前请插入手柄手柄接收器**。若在AimRT图标出现前崩溃请确认前置环境依赖是否安装完善，若在AimRT图标出现后崩溃请确认是否插入手柄。
```bash
cd build/
./run_sim.sh
```
运行成功后有窗口打开并显示机器人仿真画面。

### 1.4 手柄操控
<a id="手柄操控"></a>

注：仿真时，先将机器人切换至`ZERO`状态，然后点击`reset`让机器人站立，再切换至行走模式。
1.  启动程序后，默认状态为`idle`。
2.  按手柄上的`START`键切换到`idle`空闲状态。按手柄上的`BACK`键切换到`keep`保持状态
3.  按`B`键切换到`zero`归零状态。按`A`键切换到`stand`站立状态。按`X`键切换到`walk_leg`行走状态。按`Y`键切换到`walk_leg_arm`行走状态。
4.  在仿真界面中，点击`Reset`按钮，使机器人调整到站立准备姿态。
5.  在行走模式下，按住 LB，同时：
  - 推动`左摇杆`控制机器人行走。
  - 推动`右摇杆`控制机器人转弯。
6.  在任何状态，按 RB 键可以挥手 (需在 keep, stand, walk_leg 状态)。按 START 键可随时切回`idle`状态。
