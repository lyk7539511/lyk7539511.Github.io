---
title: "Ubuntu深度学习服务器环境配置"
categories:
  - Blog
  - Tech
tags:
  - Linux
  - Deep Learning
  - Ubuntu
---

## 1、核心硬件介绍

| 配件 | 型号 |
| --- | --- |
| CPU | R5 7600X |
| 内存 | DDR5 5200 16G*2 |
| 显卡 | TUF 3090 24G |

## 2、操作系统安装

### 2.1、系统选择

系统选择 Ubuntu Desktop 22.04 LTS

系统下载地址：[https://ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)

本教程所有配置过程均使用命令行完成。

### 2.2、系统安装

将下载好的系统镜像直接解压放入U盘即可，当前主板基本都支持UEFI启动。

使用U盘启动电脑，进入安装界面，选择语言，选择键盘布局，选择磁盘分区，设置用户名和密码，等待安装完成。

系统安装过程可参考 [https://www.bilibili.com/video/BV1Hz4y117SZ](https://www.bilibili.com/video/BV1Hz4y117SZ)

## 3、系统配置

使用你的用户名和密码登录系统，进入系统后请先更新软件，更新命令如下：

```bash
[username]@[hostname]:~$ sudo apt update
[username]@[hostname]:~$ sudo apt upgrade
```

使用命令查看当前系统IP地址：

```bash
[username]@[hostname]:~$ ip addr
```

获取IP后就可以使用SSH远程登录了，这里不对任何SSH软件做推荐，大家可以自行选择。最轻量化的就是使用系统自带的ssh命令，命令如下：

```bash
[username]@[hostname]:~$ ssh [username]@[ip]
```

如果是Windows系统请自行百度如何开启ssh服务，然后使用ssh命令登录。

建议参考以下步骤将服务器IP地址进行固定，方便后面的使用。

```bash
# 安装net-tools
[username]@[hostname]:~$ sudo apt install net-tools

# 编辑ip配置文件
[username]@[hostname]:~$ sudo vim /etc/netplan/00-installer-config.yaml

# 修改文件为以下内容
network:
  ethernets:
    eno1:
      addresses: [192.168.100.127/24] # 这里填写你的IP地址
      optional: true
      routes:
        - to: default
          via: 192.168.100.254 # 这里填写你的网关地址
      dhcp4: false # 这里关闭DHCP
      nameservers:
        addresses: [8.8.8.8, 114.114.114.114] # 这里填写你的DNS地址
  version: 2
  renderer: networkd

# 保存退出后执行以下命令
[username]@[hostname]:~$ sudo netplan apply
```

## 4、基础软件

### 4.1、安装C/C++开发环境

```bash
[username]@[hostname]:~$ sudo apt install build-essential
```

Ubuntu本身自带gcc、g++等，但是还缺少一些有可能会用到的库，使用上面的命令安装即可。

### 4.2、安装Python开发环境

#### 4.2.1、下载Miniconda

```bash
[username]@[hostname]:~$ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
[username]@[hostname]:~$ bash Miniconda3-latest-Linux-x86_64.sh # 安装过程中一路回车YES即可
```

安装完成后需要重新加载bash配置文件，命令如下：

```bash
[username]@[hostname]:~$ source ~/.bashrc
```

嫌麻烦可以直接重启系统。

此时你的命令行应该多了一个(base)标识，代表已经安装成功。
    
```bash
[username]@[hostname]:~$ # 这是默认的命令行提示符
(base) [username]@[hostname]:~$ # 这是安装成功后的命令行提示符
```

#### 4.2.2、修改conda源

修改conda配置文件，命令如下：

```bash
# 创建配置文件
(base) [username]@[hostname]:~$ conda config --set show_channel_urls yes
# 编辑配置文件
(base) [username]@[hostname]:~$ vim ~/.condarc
# 修改文件内容为以下内容
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch-lts: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  deepmodeling: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/
# 保存退出后执行以下命令
(base) [username]@[hostname]:~$ conda clean -i
```

然后就能畅快的使用conda了。

#### 4.2.3、修改pip源

修改pip配置文件，命令如下：

```bash
# 将pip升级至最新版本
(base) [username]@[hostname]:~$ python -m pip install --upgrade pip
(base) [username]@[hostname]:~$ vim ~/.pip/pip.conf
# 将阿里源设置为默认源，不推荐使用清华源，因为清华源有些包有问题
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com

```

然后就能畅快的使用pip了。

### 4.3、安装OpenSSH Server

```bash
(base) [username]@[hostname]:~$ sudo apt install openssh-server
```

### 4.4、安装Nvidia驱动

在安装之前请先禁用开源驱动，命令如下：

```bash
(base) [username]@[hostname]:~$ sudo vim /etc/modprobe.d/blacklist-nouveau.conf
# 在打开的文件中添加以下内容
blacklist nouveau
options nouveau modeset=0

# 更新设置
(base) [username]@[hostname]:~$ sudo update-initramfs -u
```

执行以下命令安装驱动：

```bash
(base) [username]@[hostname]:~$ sudo apt search nvidia-driver | grep nvidia-driver
# 在搜索结果中选择最新的版本，然后执行以下命令
(base) [username]@[hostname]:~$ sudo apt install nvidia-driver-xxx # xxx为版本号
```

安装完成后请重启系统。

运行以下命令查看驱动是否安装成功：

```bash
(base) [username]@[hostname]:~$ nvidia-smi

Mon Jul 17 20:28:19 2023
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.54.03              Driver Version: 535.54.03    CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA GeForce RTX 3090        Off | 00000000:01:00.0 Off |                  N/A |
|  0%   41C    P8              25W / 350W |     12MiB / 24576MiB |      0%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+

+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|    0   N/A  N/A      1126      G   /usr/lib/xorg/Xorg                            4MiB |
+---------------------------------------------------------------------------------------+
```
