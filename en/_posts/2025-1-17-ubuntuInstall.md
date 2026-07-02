---
layout: post
title: Switch Your Computer to Ubuntu System
subtitle: 将手中的旧电脑改为Ubuntu系统
date: 2025-1-17
author: Byolio
header-img: img/ubuntu.jpg
catalog: true
tags:
  - linux
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2025-1-17-ubuntuInstall/) | [English](https://www.byolio.blog/en/en/_posts/2025-1-17-ubuntuInstall/)

> This article mainly introduces how to switch your computer to the Ubuntu system.

## Introduction
As the usage time of a computer increases, the performance of a Windows computer gradually declines, and its efficiency decreases. Therefore, you can switch your computer to the Ubuntu system to improve its efficiency, turning it into a server or a remote computer, turning waste into treasure. Compared to the CentOS system, Ubuntu has a more visually appealing UI interface, so I chose the Ubuntu system.

## Preparation
The Ubuntu system version I chose is 24.04 LTS. According to the official minimum requirements:
1. The processor's clock speed must be at least 2GHz, with at least two cores (you can check this using Task Manager)
2. At least 4GB of RAM
3. You can install a dual system, but it requires at least 25GB of free hard drive space
![ubunturequire](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/ubunturequire.png)
Here, I use a USB drive for installation because it is convenient and fast, and after installation, the USB drive can be retained for future use.

## Installing the System
1. Download the Ubuntu system image file
Open the [official download page](https://ubuntu.com/download/desktop)
Click the `Download 24.04 LTS` button in the image to download the corresponding ISO image file
![downloadUbuntu](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/downloadUbuntu.png)
2. Download the software for creating the installation USB drive
Open the balenaEtcher [official download page](https://etcher.balena.io/#download-etcher)
![balenaEtcherInstall](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/balenaEtcherInstall.png)
I am using a Windows system, so I chose the Windows version of balenaEtcher to download and install
3. Create the USB drive
Open the balenaEtcher software
![balenaEtcher](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/balenaEtcher.png)
Click `Flash from file`, select the downloaded Ubuntu system ISO image file, choose the corresponding USB drive as the target disk, click Flash, wait for the flashing to complete, and then safely remove the USB drive
4. Install the system
Insert the USB drive into the computer where you want to install Ubuntu, power it on, use the shortcut key (which may vary depending on the computer model and brand; you need to check the official website of the corresponding brand) or advanced startup to enter the BIOS settings, set the USB drive as the first boot option, save and exit. The system will then restart the computer and enter the USB installation interface. Select `Try and Install Ubuntu` to install the system, then follow the prompts to complete the installation. After installation, the computer will automatically restart, and you can enter the system.

## FAQ
If you choose to connect to the network during installation, it may take a long time because domestic networks cannot directly connect to the Ubuntu image source. You can choose not to connect to the network and manually set the image source, which will significantly shorten the installation time.

## Summary
This article mainly introduces how to switch your computer from the Windows system to the Ubuntu system. I hope it helps you.

