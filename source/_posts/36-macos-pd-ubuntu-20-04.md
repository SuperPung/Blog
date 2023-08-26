---
url: macos-pd-ubuntu-20-04
title: M1 Mac 安装虚拟机
date: 2021-01-27 19:17:18
categories: [技术]
tags: [macOS, Parallels Desktop, Linux, Ubuntu]
---

macOS&Parallels_Desktop&Ubuntu_20.04 安装虚拟机过程记录

<!--more-->

本文使用的软件版本及系统环境：

- macOS Big Sur (arm64)
- Ubuntu 20.04.2 LTS (arm64)
- Parallels Desktop 16 for M1 Mac Technical Preview

# 下载 Ubuntu 20.04.2 arm64 iso 文件

由于大部分主流操作系统（Windows、macOS等）均为 amd64（即[x86-64](https://zh.wikipedia.org/wiki/X86-64)）架构，Ubuntu 各版本镜像文件也都以 amd64 为主。先后在[Ubuntu官网](https://cn.ubuntu.com)、[Ubuntu releases](https://releases.ubuntu.com)、[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn)、[网易开源镜像站](https://mirrors.163.com)等处均未找到 arm 版本镜像文件。

![ubuntuReleases](https://i0.hdslb.com/bfs/album/dfe69d7e4b83b56d22d16b1428e1468437059f54.png)

最后终于在[Ubuntu CDImage](http://cdimage.ubuntu.com)处找到。

访问[Ubuntu CDImage](http://cdimage.ubuntu.com)，若点击“releases/”并进入其子目录后，无法找到 arm 版本的桌面镜像，最多可以找到各版本的 arm 服务器安装镜像（这已经比[Ubuntu releases](https://releases.ubuntu.com)全面了）：

![ubuntuCdimageReleases](https://i0.hdslb.com/bfs/album/3efd4685e837a5e2308b76449126ff2aaa115c7a.png)

正确的“打开方式”是，不进入“releases/”，而是进入“focal/”-“daily-live”-“current”，此时就可以看到我们要下载的 arm 桌面镜像了：

![ubuntuCdimage](https://i0.hdslb.com/bfs/album/5c3f4dcd466de404819c6d83ff5fc9929d9e0efa.png)

点击右侧“64-bit ARM (ARMv8/AArch64) desktop image”即可下载。

或直接点此[下载链接](http://cdimage.ubuntu.com/focal/daily-live/current/focal-desktop-arm64.iso)进行下载。

> 注：下载可能需要国际网络环境。

# 下载 Parallels Desktop

访问[文章链接](https://www.parallels.com/blogs/parallels-desktop-apple-silicon-mac/)，点击“Try Technical Preview”，登录，阅读 “Step 1”、“Step 2”后，进入“Step 3”，点击“Parallels Desktop 16 for M1 Mac build 50393”右侧的“DOWNLOAD”即可下载，记录下方的 MD5 校验和和激活密钥。

或直接点此[下载链接](https://www.parallels.com/directdownload/pdbeta)进行下载。

> 注：下载可能需要国际网络环境。

下载完成后，检查 MD5 校验和：打开“终端”，键入`md5 `（注意空格）并将 dmg 文件拖至其后，得到的 MD5 校验和与 Parallels 给出的进行比较，若一致则文件完好。

# 安装 Parallels Desktop

双击 dmg 文件，按照提示即可成功安装。

# 新建虚拟机

输入激活密钥后，打开 Parallels Desktop：

![pd打开](https://i0.hdslb.com/bfs/album/e240d5f10ac469fac5d1d7bd9ef3c5ac9195678a.png)

继续，新建虚拟机：

![pd自动查找](https://i0.hdslb.com/bfs/album/2e0615f6c0f884f105415a7dce05286fda423dba.png)

点击“手动选择”：

![pd手动选择](https://i0.hdslb.com/bfs/album/42716a66659e076aff0a11ec00917f9da8da3e1a.png)

点击“选择文件…”并选择下载好的 iso 文件：

![pd新建](https://i0.hdslb.com/bfs/album/8f6394fff25b8c6d2dcdf1bd22583c6557e45e18.png)

识别为“Ubuntu Linux”后，点击“继续”开始安装。

稍等片刻即成功创建虚拟机。

# 开启虚拟机，安装 Ubuntu

开启虚拟机，选择语言为“中文(简体)”：

![pd欢迎](https://i0.hdslb.com/bfs/album/709bb60b95741ba6aa137e20b045c6546656df93.png)

继续，选择键盘布局：

![pd键盘布局](https://i0.hdslb.com/bfs/album/31a547da8d0df6856bea5f47ce6d9f3a998e52e2.png)

默认，继续，选择安装选项，取消“安装 Ubuntu 时下载更新”以节约安装的时间：

![pd取消更新](https://i0.hdslb.com/bfs/album/e13d0db78286d5b56de3fc0e1fc778a36ff1f98e.png)

继续，选择安装类型“清除整个磁盘并安装 Ubuntu”：

![pd安装类型](https://i0.hdslb.com/bfs/album/2f25f57b52ba124e306203d51f566c8667b5cf2c.png)

现在安装，将改动写入磁盘：

![pd写入磁盘](https://i0.hdslb.com/bfs/album/8b6e92a7f72404fd66f2431a114af52a348fb21f.png)

继续选择地区为“Shanghai”：

![pd上海](https://i0.hdslb.com/bfs/album/eda913d0eafa86838150b11343ecc3a11f37389e.png)

继续设置姓名：

![pd姓名](https://i0.hdslb.com/bfs/album/8d05e09d6d6800a906e4e61791de520e26fec6b3.png)

输入姓名、计算机名、用户名及密码后，继续进入安装页面，等待：

![pd安装等待](https://i0.hdslb.com/bfs/album/1711b441f651e25f17f92f2561da6cc93d3b4107.png)

安装成功后，进入选择在线账号：

![pd在线账号](https://i0.hdslb.com/bfs/album/ed6b5849f45374bc64b00ffc56f0d4fcbead355d.png)

跳过即可，Livepatch：

![pdLivepatch](https://i0.hdslb.com/bfs/album/177a25e1236347c2096a5c73a5b2c1d381b22ccb.png)

隐私设置：

![pd隐私](https://i0.hdslb.com/bfs/album/8a9122f2ae41cfa6b7749b2ad275f9b466e0d17e.png)

帮助改进 Ubuntu：

![pd帮助改进](https://i0.hdslb.com/bfs/album/bf0a4dfe5eb078081b3b80e8a15323505ca9a093.png)

准备就绪：

![pd准备就绪](https://i0.hdslb.com/bfs/album/b5a4fa520e111bf4729412067606f8c9fb57f9a0.png)

安装完成：

![pd安装完成](https://i0.hdslb.com/bfs/album/6e9acb24aed462e39ca2fa7a40ef25b14f3b9282.png)
