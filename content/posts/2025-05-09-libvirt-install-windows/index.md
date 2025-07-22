---
title: 在ArchLinux下使用QEMU/KVM和Virt Manager安装Windows虚拟机
description: 以及解决虚拟机无网络的问题
date: 2025-05-09
tags: [Linux, 虚拟机, Debug]
---

## 安装系统前的准备

参考：https://ivonblog.com/posts/archlinux-qemu-virt-manager/

安装必要的包

```bash
sudo pacman -S qemu-full libvirt virt-manager
```

还有一些utils包，缺啥装啥

```bash
sudo pacman -S dnsmasq 
```

启动服务

```bash
sudo systemctl enable --now libvirtd
sudo virsh net-start default
sudo virsh net-autostart default
```

把当前用户加到用户组

```bash
sudo gpasswd -a $USER libvirt
sudo gpasswd -a $USER kvm
```

## 安装Windows 10系统

参考：https://ivonblog.com/posts/install-windows-11-qemu-kvm-on-linux/

### 问题排查：无网络

我使用的是NAT模式（懒得配置桥接了），网卡选择了virtio模式。问题在于virtio模式也需要安装特定驱动，否则windows上无法识别网卡。

DeepSeek表示：

> 在下载并挂载了virtio-win.iso之后：
> 
> 在Windows中安装驱动​​：打开设备管理器 → 右键未知设备 → 更新驱动程序 → 浏览ISO中的NetKVM目录（根据系统选择w10或w11）。

## 代理设置

宿主机代理软件开启port sharing，然后通过宿主机运行`ip addr`或虚拟机运行`ipconfig`获取虚拟机的默认网关，最后在虚拟机设置中填入到代理服务器即可。

## 使用VirtioFS共享文件夹

参考：https://ivonblog.com/posts/libvirt-virtiofs/

## TODO

* 窗口分辨率的设置
* 经常用着用着就死机了，好像也没啥好办法 :innocent: