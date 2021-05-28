+++
title = "WSL2 下 Arch Linux 安装配置备忘"
date = "2021-05-28"

[taxonomies]
categories = ["blog"]
tags = ["Arch", "WSL"]
+++

### 起因

去年十一月 DIY 了一台 Ryzen 3700X 的机器，Windows 10 打游戏 + WSL2 跑编译。

当时微软的 WSL2 专用 Linux 内核还停留在 4.x 版本，最近发现微软升级了 5.x 版本的内核，于是在这台机器上进行了内核升级，然后 WSL2 就炸了 =_=|||。

具体的现象是，`/usr/bin` 下面一部分可执行文件的大小变成了 0 KB，数据莫名其妙丢掉了，直觉上像是 ext4 的问题，具体原因有待查证。

总之过去的系统基本报废，既然需要重装的话，这里索性把 WSL2 下安装配置 Arch Linux 的步骤记录在此，以作备忘。

### 基础配置

#### Windows 启用 WSL2

#### WSL2 配置

#### Arch 基础配置

### WSL hacks

#### 在 WSL2 环境下使用 systemd

#### 宿主机 IP 获取

#### 内存占用问题

#### 磁盘占用问题
