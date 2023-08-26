---
url: win-vm-ubuntu-20-04
title: Windows 安装 Ubuntu
date: 2021-01-27 11:47:34
categories: [技术]
tags: [VMware, Linux, Windows, Ubuntu]
---

Windows&VMware&Ubuntu_20.04 安装虚拟机过程记录

<!--more-->

本文使用的软件版本及系统环境：

- Windows 10 (amd64)
- Ubuntu 20.04.1 LTS (amd64)
- VMware Workstation Pro 16.1.0

# 下载 Ubuntu 20.04.1 amd64 iso文件

访问[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn)，点击“ubuntu-releases”-“20.04/”-“ubuntu-20.04.1-desktop-amd64.iso”即可下载。

或直接点此[下载链接](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/20.04/ubuntu-20.04.1-desktop-amd64.iso)进行下载。

# 下载 VMware Workstation Pro

访问[VMware中文官网](https://www.vmware.com/cn.html)，点击“Menu”-“下载”-“产品下载”-“Workstation Pro”：

![vmware网站](https://i0.hdslb.com/bfs/album/560f1eacfcfa7061703d67b733f73e41d5255eb9.png)

选择版本后，点击右侧“下载免费试用版：Windows”，即可无需登录直接下载。

# 安装 VMware Workstation Pro

选择安装目标及任何其他功能：

![vm安装1](https://i0.hdslb.com/bfs/album/23b56058747d9145ef73cd5bb1f5e6f2a7e3dca8.png)

输入 VMware Workstation 16 的许可证密钥：

![vm安装许可证](https://i0.hdslb.com/bfs/album/8eaad1b1e988e81c04c6e675c56c62e3f497a125.png)

可选择以下密钥之一输入：

> FG78K-0UZ[15-085](tel:15-085)TQ-TZQXV-XV0CD
>
> ZA11U-DVY97-M81LP-4MNEZ-X3AW0
>
> YU[102-44](tel:102-44)D86-48D2Z-Z4Q5C-MFAWD

安装成功。

# 新建虚拟机

打开 VMware Workstation 16，点击“创建新的虚拟机”，打开新建虚拟机向导：

![新建虚拟机向导](https://i0.hdslb.com/bfs/album/8bca47307208017cfa0f29889a20673a4fca756e.png)

选择“自定义(高级)”，继续选择虚拟机硬件兼容性：

![硬件兼容性](https://i0.hdslb.com/bfs/album/e8bfbbd9936af6b5a4c0532c3c20e0b5438d8d13.png)

默认，进行下一步，选择客户机操作系统：

![客户机操作系统](https://i0.hdslb.com/bfs/album/0acf061340d2dabe264f8936799b5d348c6ec913.png)

选择“Microsoft Windows”，版本“Windows 10 x64”，进行下一步，安装客户机操作系统：

![安装客户机操作系统](https://i0.hdslb.com/bfs/album/54a6a8eee89120992397cb0815c36ebf5f50be6b.png)

选择“稍后安装操作系统”，进行下一步，选择客户机操作系统：

![选择客户机操作系统](https://i0.hdslb.com/bfs/album/496002920037d911cecb25ef3966c7d248445056.png)

选择“Linux”，版本“Ubuntu 64 位”，进行下一步，命名虚拟机：

![命名虚拟机](https://i0.hdslb.com/bfs/album/06f86a9ed844a6f6c2ae194063fc0ca9468e2cd9.png)

设置虚拟机名称和位置后，进行下一步，处理器配置：

![处理器配置](https://i0.hdslb.com/bfs/album/1f74553c7228772de041fdf4ed58ceb5424f3ce5.png)

默认分配 2 个处理器、每个处理器 1 个内核，进行下一步，内存配置：

![内存](https://i0.hdslb.com/bfs/album/7d15c3ab6978621a043dc8b51f73f3e41b7ede8a.png)

分配 4GB 内存，进行下一步，选择网络类型：

![网络类型](https://i0.hdslb.com/bfs/album/efd0ed3620c0c00b1c1bafe225924ddd797b5100.png)

默认选择“使用网络地址转换(NAT)”，进行下一步，选择 I/O 控制器类型：

![IO控制器类型](https://i0.hdslb.com/bfs/album/0cbc4d5cdd574b5617e7b82e14118e0aee62f878.png)

默认选择“LSI Logic”，进行下一步，选择磁盘类型：

![磁盘类型](https://i0.hdslb.com/bfs/album/fc3da892c6b593a2820fb8157527e11391299dc3.png)

默认选择“SCSI”，进行下一步，选择磁盘：

![选择磁盘](https://i0.hdslb.com/bfs/album/f151487de46aac5fa6647698245e7f532a48cd43.png)

默认选择“创建新虚拟磁盘”，进行下一步，指定磁盘容量：

![指定磁盘容量](https://i0.hdslb.com/bfs/album/f0a018da710e79ee96dde9984e39fc31ad376cec.png)

将最大磁盘大小设置为 30.0GB，默认选择“将虚拟磁盘拆分成多个文件”：

![指定磁盘容量30](https://i0.hdslb.com/bfs/album/0d5613d00f92ae854caf581faaea9a0f65f04929.png)

进行下一步，指定磁盘文件，选择存储位置后，创建完成：

![已准备好创建虚拟机](https://i0.hdslb.com/bfs/album/1ceb210b6f6ebeda892fc2b664f9f1aff25df488.png)

点击“完成”，编辑虚拟机设置：

![虚拟机设置](https://i0.hdslb.com/bfs/album/2afbde8b29b3a3f04452d01f68053864551d8e29.png)

点击“CD/DVD (SATA)”，在右侧“连接”处选择“使用 ISO 映像文件”，选择下载好的 iso 文件：

![虚拟机设置cddvd](https://i0.hdslb.com/bfs/album/c25afdfd97ebaee901169c2c9f3d6a2bc5f961a7.png)

“确定”。

# 开启虚拟机，安装 Ubuntu

![ubuntu安装1](https://i0.hdslb.com/bfs/album/8ea0a7a6e510b062aca4fd962011a14c6f848e43.png)

等待……

![ubuntu安装2](https://i0.hdslb.com/bfs/album/a0ecd7c53749814bc8279c2b458da516fb45aa56.png)

选择语言：

![ubuntu安装选择语言](https://i0.hdslb.com/bfs/album/eca2a8324d3ff7e031c31b0d9bdc4ffc9df29f74.png)

选择中文(简体)：

![ubuntu安装选择语言中文](https://i0.hdslb.com/bfs/album/79db6fc0a677a6a1cb48ef331c379a6b88b54670.png)

点击“安装 Ubuntu”，选择键盘布局：

![ubuntu安装选择键盘布局](https://i0.hdslb.com/bfs/album/5d85f9dd08c5ea80d37e6f6d18d7bb45c88c9bc9.png)

默认，继续，选择安装选项：

![ubuntu安装更新](https://i0.hdslb.com/bfs/album/c110d12d1cd072e88be16ac5a34406223b655c45.png)

取消“安装 Ubuntu 时下载更新”以节约安装的时间：

![ubuntu安装取消更新](https://i0.hdslb.com/bfs/album/feb029a0ee2da3134ed76e4ee09fdce7d3637324.png)

继续，选择安装类型“Erase disk and install Ubuntu”后，继续将改动写入磁盘：

![ubuntu安装选择磁盘](https://i0.hdslb.com/bfs/album/fed9000fe86cd082b790f9c6a550d857ec155d64.png)

继续选择地区：

![ubuntu安装您在什么地方](https://i0.hdslb.com/bfs/album/30a99d70eadc36be81ec9a426c55b1dbbdb4b6f5.png)

选择“Shanghai”：

![ubuntu安装在上海](https://i0.hdslb.com/bfs/album/d81cdea6a8fd2b2b40325fb3cae12a56d179fd2b.png)

继续设置姓名：

![ubuntu安装您是谁](https://i0.hdslb.com/bfs/album/9fdc9e5c9008849fec14e4031a6ca4739f91764e.png)

输入姓名、计算机名、用户名及密码：

![ubuntu安装姓名](https://i0.hdslb.com/bfs/album/91d0616b749a1ddf271c393a3789715667358e98.png)

继续进入安装页面，等待：

![ubuntu安装等待](https://i0.hdslb.com/bfs/album/0c083da0b9c37a20b9003412c3c9af3e7723bcab.png)

安装成功后，进入选择在线账号：

![ubuntu安装在线账号](https://i0.hdslb.com/bfs/album/254749064ba908cd06438b10526e5349b7d70a0b.png)

跳过即可，Livepatch：

![ubuntu安装livepatch](https://i0.hdslb.com/bfs/album/8c6fcc149c02346c54b162ce75148ec6316be10a.png)

帮助改进 Ubuntu：

![帮助改进ubuntu](https://i0.hdslb.com/bfs/album/6f929f84b3ecec15159a6eaeab94ba6d9db30bc3.png)

准备就绪：

![ubuntu准备就绪](https://i0.hdslb.com/bfs/album/44f310282174d7a62ff91fdd7d5b2f87ffee93d5.png)

安装完成：

![ubuntu安装完成](https://i0.hdslb.com/bfs/album/6af1cdc79fcf978597dfff7a83f476e311b2ab6c.png)
