---
layout:     post
title:      "将电脑改为Ubuntu系统"
subtitle:   "将手中的旧电脑改为Ubuntu系统"
date:       2025-1-17
author:     "Byolio"
header-img: "img/ubuntu.jpg"
catalog: true
tags:
    - linux
---
> 本文主要介绍了如何将电脑改为Ubuntu系统。

## 引言
随着电脑的使用时间的增加, windows电脑的性能逐渐下降, 使用效率逐渐降低, 因此可以将电脑改为Ubuntu系统, 以提高电脑的使用效率, 成为一个服务器或远程电脑, 变废为宝, 而ubuntu相比于centos系统界面的UI比较好看, 因此我选择了ubuntu系统。

## 准备工作
我选用的ubuntu系统版本为24.04LTS版, 按照官网最低要求如下:
1. 处理器的主频至少需要达到2GHz，并且至少有两个核心(可以使用任务管理器查看)
2. 最少有4GB的内存
3. 可以安装双系统, 但要求剩余硬盘空间至少有25GB以上
![ubunturequire](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/ubunturequire.png)
这里我使用U盘安装, 因为U盘安装速度方便, 而且U盘安装后可以保留U盘, 方便后续使用。

## 安装系统
1. 下载Ubuntu系统镜像文件
打开[官网下载地址](https://ubuntu.com/download/desktop)
点击图片中的`Download 24.04 LTS`按钮下载对应的iso镜像文件
![downloadUbuntu](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/downloadUbuntu.png)
2. 下载制作安装U盘的软件
打开balenaEtcher[官网下载地址](https://etcher.balena.io/#download-etcher)
![balenaEtcherInstall](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/balenaEtcherInstall.png)
我使用的是windows系统, 因此选择windows版本的balenaEtcher进行下载, 下载后进行安装
3. 制作U盘
打开balenaEtcher软件
![balenaEtcher](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/balenaEtcher.png)
点击`从文件烧录`, 选择下载好的Ubuntu系统iso镜像文件, 目标磁盘选择对应U盘, 点击烧录, 等待烧录成功并拔出U盘
4. 安装系统
将U盘插入需要安装ubuntu系统的电脑, 开机, 使用快捷键(不同型号和牌子的电脑会有所不同, 需要到对应牌子电脑的官网查看)或高级启动进入BIOS设置, 将U盘设置为第一启动项, 保存并退出, 然后系统会重启电脑, 进入U盘安装界面, 选择`Try and Install Ubuntu`安装系统, 然后根据提示进行安装, 安装完成后会自动重启电脑, 就可以进入系统了

## FAQ
安装过程如果选择链接网络, 因为国内无法直接链接ubuntu镜像源, 因此会导致安装时间较长, 可以选择不链接网络, 然后手动设置镜像源, 安装时间会大大缩短。

## 总结
本文主要介绍了如何将电脑windows系统改为Ubuntu系统, 希望对你有所帮助。

