+++
title = "WSL2 下 Arch Linux 安装配置备忘"
date = "2021-05-28"

[taxonomies]
categories = ["blog"]
tags = ["Arch", "WSL"]
+++

### 起因

去年显卡价格疯涨前夕 DIY 了一台 3700X 台式机，用于 Windows 10 打游戏 + WSL2 跑编译。
当时 WSL2 Linux 内核还停留在 4.x 版本，而今年微软发布了 [5.x 版本的新内核](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)。

发现 WSL2 内核版本升级了之后，我直接去升级了内核，然而升级完毕 WSL2 就炸了 =_=|||。
具体的现象是，`/usr/bin` 下面一些可执行文件的大小变成了 0 KB 丢失了数据，猜测是 ext4 的问题，具体原因有待查证。

总之 Arch 系统基本已经报废，既然需要重新配置的话，这里索性把 WSL2 下安装配置 Arch Linux 的步骤记录在此以作备忘。

### 基础配置

#### WSL2 安装配置

操作步骤参考微软官方的[指南](https://docs.microsoft.com/en-us/windows/wsl/install-win10)，WSL2 的安装步骤如下：

1. 启用所需的 Windows 功能，包括 Linux 子系统、虚拟机平台。

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
shutdown /r /t 0
```

2. 安装 [WSL2 Linux 内核](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)。

3. WSL 切换默认版本。

```powershell
wsl --set-default-version 2
```

4. 修改 WSL 配置文件 `%UserProfile%\.wslconfig`。

```ini
[wsl2]
swap=0
localhostForwarding=true
nestedVirtualization=true
```

至此 WSL2 的安装配置完成，接下来需要在 WSL2 上装一个 Linux 发行版。

#### Arch 安装配置

微软商店提供了 Ubuntu、Debian 等发行版，可以直接在应用商店下载安装。
不过，由于证书原因 Arch Linux 没有在微软商店上架，所以这里需要手工下载安装。

[ArchWSL](https://github.com/yuk7/ArchWSL) 提供了安装器和 appx 两种安装方式，除此之外还可以手工 import 发行版的 tar 包，后者可以控制安装位置等参数。
本文使用 import 方式安装 Arch Linux：

1. 下载[最新版 Arch.zip](https://github.com/yuk7/ArchWSL/releases/latest/download/Arch.zip)，提取其中的 `rootfs.tar.gz` 文件，然后再用 [7-zip](https://www.7-zip.org/) 把 `rootfs.tar.gz` 解压为 `rootfs.tar` 让 `wsl --import` 能够识别。

2. 执行命令 `wsl --import Arch <安装目录> rootfs.tar` 将发行版 `Arch` 导入到指定的安装目录下。等待导入成功后，安装目录下会生成名为 `ext4.vhdx` 的虚拟机磁盘镜像。

3. 执行命令 `wsl -d Arch -u root` 以 root 身份登录已导入的发行版中，或者直接直接执行 `wsl` 以默认身份进入默认发行版。
首先编辑 [`/etc/wsl.conf` 配置文件](https://docs.microsoft.com/en-us/windows/wsl/wsl-config#configuration-options)：

```ini
[user]
default=root

[network]
generateHosts = true
generateResolvConf = true

[interop]
enabled = true
appendWindowsPath = false
```

4. 配置软件源，安装基础开发工具。

```bash
# 初始化 pacman
pacman-key --init
pacman-key --populate

# 启用中科大镜像
perl -i -lpe 's|#(?=Server = https://mirrors.ustc.edu.cn/)||' /etc/pacman.d/mirrorlist
pacman -Syu

# 安装常用的系统工具、网络工具、开发工具
pacman -S sudo curl wget aria2 nano vim \
  bash-completion fish man-db pacman-contrib pkgfile \
  openssh mosh tmux ntp net-tools bind-tools gnu-netcat iftop \
  base-devel cmake git docker go
pacman -Scc

# 下载 pkgfile 索引，用来在缺命令时提示需要安装什么软件包
pkgfile -U
```

5. 设置用户。

```bash
# 设置 root 密码
passwd root

# 新建用户，设置用户组和密码
useradd 用户名
gpasswd -a 用户名 wheel
passwd 用户名

# 允许在 wheel 组里的用户使用 sudo
echo '%wheel ALL=(ALL) ALL' > /etc/sudoers.d/20-wheel

# 建立用户主目录，设置权限
mkdir -p ~用户名
chmod 0755 ~用户名
chown 用户名:用户名 ~用户名

# 登录 shell 切成 fish，看个人喜好
chsh -s /bin/fish 用户名

# 配置 ssh 密钥，设置正确的文件权限
mkdir -p ~用户名/.ssh
# <这里可以在 Windows 下访问 \\wsl$\Arch\home\用户名\.ssh 路径，把 ssh 密钥拷贝进来>
chmod 0700 ~用户名/.ssh
chmod 0600 ~用户名/.ssh/*
chown -R 用户名:用户名 ~用户名/.ssh
```

如果没有使用 systemd 的需求（见下文，需要以 root 身份登录来做一个 hack），到了这一步就可以直接去修改 `/etc/wsl.conf` 里的默认登录用户配置了。

6. 启用 archlinuxcn 软件源，安装 yay 工具。

```bash
# 配置 archlinuxcn 软件源的中科大镜像，MySQL 8.0 之类的软件包可以在这个仓库下载
cat >> /etc/pacman.conf <<'EOF'
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
EOF
pacman -Sy archlinuxcn-keyring

# archlinuxcn 提供的 yay 版本较旧，这里直接从 AUR 编译安装
# 首先，makepkg 不允许 root 身份进行编译打包操作，需要先切到非 root 用户上进行后续操作
sudo -u 用户名 fish
cd ~
# 其次，yay 依赖于 golang.org/x/，在国内编译时必定网络连接失败，这里要设置 GOPROXY
export GOPROXY=https://goproxy.io,direct
go env -w GOPROXY="$GOPROXY"
# 最后，下载 PKGBUILD 编译安装
git clone https://aur.archlinux.org/yay.git && cd yay
makepkg -si
cd .. && rm -rf yay
exit
```

### WSL hacks

#### 在 WSL2 环境下使用 systemd

#### 宿主机 IP 获取

#### 内存占用问题

#### 磁盘占用问题

#### 内网穿透和 sshd

```bash

# 设置 sshd 保持连接
sudo sed -i 's|#ClientAliveInterval 0|ClientAliveInterval 30|' /etc/ssh/sshd_config
```