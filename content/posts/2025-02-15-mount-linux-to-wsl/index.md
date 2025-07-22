---
title: 如何在dual-boot下共享Linux与WSL的文件
description: 省流：挂载硬盘
date: 2025-02-15
tags: [Linux]
---

## 动机

众所周知，WSL2与WSL1相比，最大的劣势就在于跨文件系统的读写效率[^1]，所以如果在WSL下进行开发，那么尽量将文件存储于WSL文件系统中。但我的笔记本是双系统，主要工作环境是Arch Linux系统，安装WSL只是预防Arch Linux滚挂的紧急情况。

我以前使用的方案是，将所有文件存储在Windows系统中，Arch Linux系统通过挂载硬盘的方式访问，WSL通过跨文件系统的方式访问。这样虽然可以实现一份文件多处共享，但受限于WSL2跨文件系统读写效率低下的问题。

因此，在购入新电脑后，我将所有文件存储在Arch Linux系统中，在WSL2中直接挂载Arch Linux系统所在的硬盘，通过挂载硬盘的方式直接访问。至于Windows系统，反正没有配置开发环境，能用编辑器打开代码文件即可。

## 操作流程

参考这两个链接一步一步来即可，具体过程略。

[Is it possible to automount partition/drive using wsl --mount after restart? · Issue #6073 · microsoft/WSL](https://github.com/microsoft/WSL/issues/6073#issuecomment-1774085064)

[Automatically Starting an External Encrypted SSD in Windows Subsystem (WSL) | by Stefan Berkner | Medium](https://medium.com/@stefan.berkner/automatically-starting-an-external-encrypted-ssd-in-windows-subsystem-wsl-6403c34e9680)

## 实验结果

我测试了三种情况下的读写实验结果。

1. WSL内部文件系统（文件存放于WSL文件系统中）

```bash
$ time dd if=/dev/zero of=/tmp/test bs=8k count=1000000

1000000+0 records in
1000000+0 records out
8192000000 bytes (8.2 GB, 7.6 GiB) copied, 14.5304 s, 564 MB/s
dd if=/dev/zero of=/tmp/test bs=8k count=1000000  0.40s user 14.10s system -45% cpu -32.143 total

$ time dd if=/tmp/test of=/dev/null bs=8k

1000000+0 records in
1000000+0 records out
8192000000 bytes (8.2 GB, 7.6 GiB) copied, 11.3272 s, 723 MB/s
dd if=/tmp/test of=/dev/null bs=8k  0.38s user 10.94s system 99% cpu 11.322 total
```

2. 跨文件系统（文件存放于Windows文件系统中）

```bash
$ time dd if=/dev/zero of=/mnt/c/Users/tangy/test bs=8k count=1000000

1000000+0 records in
1000000+0 records out
8192000000 bytes (8.2 GB, 7.6 GiB) copied, 221.01 s, 37.1 MB/s
dd if=/dev/zero of=/mnt/c/Users/tangy/test bs=8k count=1000000  1.30s user 14.70s system 9% cpu 2:54.32 total

$ time dd if=/mnt/c/Users/tangy/test of=/dev/null bs=8k

1000000+0 records in
1000000+0 records out
8192000000 bytes (8.2 GB, 7.6 GiB) copied, 256.3 s, 32.0 MB/s
dd if=/mnt/c/Users/tangy/test of=/dev/null bs=8k  1.28s user 12.21s system 5% cpu 4:16.30 total
```

3. 挂载硬盘（文件存放于挂载硬盘的Arch Linux文件系统中）

```bash
$ time dd if=/dev/zero of=/mnt/wsl/PHYSICALDRIVE1p3/@home/adswt518/test bs=8k count=1000000

1000000+0 records in
1000000+0 records out
8192000000 bytes (8.2 GB, 7.6 GiB) copied, 10.8812 s, 753 MB/s
dd if=/dev/zero of=/mnt/wsl/PHYSICALDRIVE1p3/@home/adswt518/test bs=8k   0.30s user 10.58s system 18% cpu 57.570 total

$ time dd if=/mnt/wsl/PHYSICALDRIVE1p3/@home/adswt518/test of=/dev/null bs=8k

1000000+0 records in
1000000+0 records out
8192000000 bytes (8.2 GB, 7.6 GiB) copied, 2.04829 s, 4.0 GB/s
dd if=/mnt/wsl/PHYSICALDRIVE1p3/@home/adswt518/test of=/dev/null bs=8k  0.22s user 1.83s system 99% cpu 2.053 total
```

可以看到1和3的效率远高于2，说明WSL2跨文件系统的读写效率确实很烂。而3比1的效率更高，可能是因为我自己买来安装Arch Linux的硬盘比电脑出厂自带的安装Windows的硬盘更好。

[^1]: https://learn.microsoft.com/en-us/windows/wsl/compare-versions