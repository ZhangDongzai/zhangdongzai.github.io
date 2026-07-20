---
title: 在 Termux 中使用 chroot 部署 Ubuntu
date: 2026-07-20 16:55:34
tags: [Linux, Ubuntu, Termux]
---

# 环境准备
## 设备 && 软件
1. 一部已经 Root 的手机（推荐Magisk）
2. 安装[Termux](https://github.com/termux/termux-app/releases)
3. 安装[Busybox](https://github.com/Magisk-Modules-Alt-Repo/BuiltIn-BusyBox/releases)模块

## 更新 && 下载软件包
```sh
pkg update && pkg upgrade  
pkg install root-repo  
pkg install tsu wget vim
```

## 切换到 /data/local/tmp 目录
在这个目录中可以避免权限问题
```sh
su
cd /data/local/tmp
```

## 下载 Ubuntu 24.04 LTS 的 rootfs 文件
```sh
wget http://cdimage.ubuntu.com/ubuntu-base/noble/daily/current/noble-base-arm64.tar.gz
mkdir chroot
tar xpf noble-base-arm64.tar.gz -C chroot
mkdir ./chroot/sdcard
mkdir ./chroot/dev/shm
```

# 启动
创建`start-chroot.sh`脚本
```sh
#!/bin/sh  

UBUNTUPATH="/data/local/tmp/chroot"  

busybox mount -o remount,dev,suid /data  
busybox mount --bind /dev $UBUNTUPATH/dev  
busybox mount --bind /sys $UBUNTUPATH/sys  
busybox mount --bind /proc $UBUNTUPATH/proc  
busybox mount -t devpts devpts $UBUNTUPATH/dev/pts  
  
busybox mount -t tmpfs -o size=256M tmpfs $UBUNTUPATH/dev/shm  
  
busybox mount --bind /sdcard $UBUNTUPATH/sdcard  
  
busybox chroot $UBUNTUPATH /bin/su - root  
  
busybox umount $UBUNTUPATH/dev/shm  
busybox umount $UBUNTUPATH/dev/pts  
busybox umount $UBUNTUPATH/dev  
busybox umount $UBUNTUPATH/proc  
busybox umount $UBUNTUPATH/sys  
busybox umount $UBUNTUPATH/sdcard
```
赋予执行权利并启动
```sh
chmod +x start-chroot.sh
./start-chroot.sh
```

# 设置网络
在容器内设置
```sh
echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "127.0.0.1 localhost" > /etc/hosts

groupadd -g 3003 aid_inet
groupadd -g 3004 aid_net_raw
groupadd -g 1003 aid_graphics
usermod -g 3003 -G 3003,3004 -a _apt
usermod -G 3003 -a root
```
在Termux中设置
```sh
exit
vim ./chroot/etc/passwd
```
把`_apt:x:42:***`改为`_apt:x:0:***`



# 鸣谢
[Ivon的部落格](https://ivonblog.com/)的文章[[Root] 手機Termux建立chroot Ubuntu環境，免Linux Deploy](https://ivonblog.com/posts/termux-chroot-ubuntu/)
[CSY2022](https://github.com/CSY2022)的文章[在 Termux 上利用 chroot 运行 Ubuntu](https://blog.csy2022.top/2025/%E5%9C%A8%20Termux%20%E4%B8%8A%E5%88%A9%E7%94%A8%20chroot%20%E8%BF%90%E8%A1%8C%20Ubuntu)
[锁部千本](https://space.bilibili.com/1704672578)的文章[【开发记录】手机搭建服务器环境——linuxdeploy安装非内置版本linux](https://www.bilibili.com/opus/735435144434810905)
