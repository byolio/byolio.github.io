---
layout: post
title: Convert Your Computer to Ubuntu System
subtitle: 将手中的旧电脑改为Ubuntu系统
date: 2025-1-17
author: Byolio
header-img: img/ubuntu.jpg
catalog: true
tags:
  - linux
lang: en
---

> 🌐 [中文](/2025-1-17-ubuntuInstall/) | [English](/en/en/_posts/2025-1-17-ubuntuInstall/)

> This article mainly introduces how to convert your computer to an Ubuntu system.

## Introduction
As the usage time of a computer increases, the performance of a Windows computer gradually declines, and its efficiency decreases. Therefore, you can convert your computer to an Ubuntu system to improve its efficiency, turning it into a server or a remote computer, turning waste into treasure. Compared to the CentOS system, Ubuntu has a more attractive UI, so I chose the Ubuntu system.

## Preparation
I chose Ubuntu 24.04 LTS. According to the official website, the minimum requirements are as follows:
1. The processor must have a clock speed of at least 2 GHz and at least two cores (you can check this using Task Manager).
2. At least 4 GB of RAM.
3. You can install a dual-boot system, but at least 25 GB of free hard disk space is required.
![ubunturequire](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/ubunturequire.png)
Here, I use a USB drive to install, because it is convenient and fast, and after installation, the USB drive can be retained for future use.

## Installing the System
1. Download the Ubuntu system ISO file.
Open the [official download page](https://ubuntu.com/download/desktop)
Click the `Download 24.04 LTS` button in the image to download the corresponding ISO file.
![downloadUbuntu](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/downloadUbuntu.png)
2. Download the software for creating a bootable USB drive.
Open balenaEtcher [official download page](https://etcher.balena.io/#download-etcher)
![balenaEtcherInstall](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/balenaEtcherInstall.png)
I am using a Windows system, so I choose the Windows version of balenaEtcher to download, and then install it.
3. Create the bootable USB drive.
Open the balenaEtcher software.
![balenaEtcher](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/balenaEtcher.png)
Click `从文件烧录`, select the downloaded Ubuntu ISO file, choose the target disk as the USB drive, click Flash, wait for the flashing to complete, and then remove the USB drive.
4. Install the system.
Insert the USB drive into the computer where you want to install Ubuntu, power on, use a shortcut key (which varies by computer model and brand; you need to check the official website of the corresponding brand) or advanced startup to enter the BIOS settings, set the USB drive as the first boot device, save and exit. Then the system will restart, enter the USB installation interface, select `Try and Install Ubuntu` to install the system, and follow the prompts to complete the installation. After installation, the computer will automatically restart, and you can enter the system.

## FAQ
If you choose to connect to the network during installation, because the Ubuntu mirror source cannot be directly accessed from within China, the installation time will be longer. You can choose not to connect to the network and manually set the mirror source, which will significantly shorten the installation time.

## Summary
This article mainly introduced how to convert your computer from Windows to Ubuntu. I hope it helps you.

