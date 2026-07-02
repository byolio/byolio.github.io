---
layout: post
title: How to Deploy a Hadoop Cluster (Part 1)
subtitle: 让我们通过vmware虚拟机来简单部署hadoop3.x集群
date: 2025-2-23
author: Byolio
header-img: img/hadoopdeploy.jpg
catalog: true
tags:
  - linux
  - hadoop
lang: en
---

> 🌐 [中文](/2025-2-23-hadoopdeploy/) | [English](/en/en/_posts/2025-2-23-hadoopdeploy/)

> This article mainly introduces the process of setting up virtual machines required for a Hadoop 3.x cluster.

## What is Hadoop
Hadoop is an open-source distributed computing framework developed by the Apache Foundation, used for storing and analyzing large-scale datasets. It is a reliable, scalable distributed computing ecosystem that allows users to process massive amounts of data on a cluster of multiple computers using a simple programming model. It is one of the core components of big data.

## Why Deploy a Hadoop Cluster
In the era of big data, data volume is growing exponentially, and traditional single-machine processing can no longer meet the demand. A Hadoop cluster can store data dispersedly across multiple computers and process massive data through parallel computing, thereby improving data processing efficiency and speed.

## What to Prepare for Deploying a Hadoop Cluster
1. VMware 17.5 or above. It is not recommended to install other 17.x versions, as VMware has warned about USB security vulnerabilities in those versions.
2. JDK and a compatible Hadoop 3.x version.
3. CentOS 7 ISO image file.
4. Since this blog will build a cluster with 4 virtual machines, it is recommended to have at least `16G` of memory, an SSD with at least `160G` of free space, and a 16-core CPU (each VM: 4G+ memory, 40G+ storage, 4+ cores).
5. Some Linux knowledge.

## Deploying a Hadoop Cluster
The following describes how to deploy a Hadoop 3.x cluster in VMware virtual machines (please ensure you have installed VMware 17.5 or above):
### 1. Configure the CentOS System
Open VMware, click "Create a New Virtual Machine", select "Custom (Advanced)" -> the corresponding VMware version -> "I will install the operating system later" -> Select "Linux", version "Red Hat Enterprise Linux 7 64-bit" (for CentOS 7.x) -> Set the installation name, location, number of cores, memory size, network type NAT, storage size, and choose to store the virtual disk as a single file or split into multiple files (select SCSI controller and SCSI hard disk), then click "Finish".
Then open the virtual machine properties, select "CD/DVD", choose "Use ISO image file", select the downloaded CentOS 7 ISO image file, ensure the network adapter's network connection mode is NAT, and click "OK".
### 2. Install the CentOS System
Power on the virtual machine, select language and keyboard layout, then enter the installation interface. Proceed to software selection, choose "Minimal Install" (no GUI) or "GNOME Desktop" (with GUI, recommended for beginners), then click "OK".
* Note: If you choose GNOME Desktop, you can select options like "Compatibility Libraries", "Legacy X Windows System Compatibility", "Development Tools", etc.

Open "Installation Destination", select "I will configure partitioning", click "Done", then add partition sizes for the hard disk, and click "Done".
* Note: Set the `/boot` partition size to `1G` or more with filesystem `ext4`, the `swap` partition size to `2G` or more with filesystem `swap`, and the `/` partition size to the remaining space with filesystem `ext4`.

Configure the hostname and other settings, then click "Begin Installation".
* Note: If your memory is limited, you can disable kdump during learning to save some memory.
Set the root password and user password, wait for the installation to complete, reboot, accept the license, and finish the CentOS system installation.

### 3. Configure VMware NAT Network Mode
Click "Edit" on VMware, then "Virtual Network Editor", click "Change Settings" -> "Add Network", select "vmnet8", choose "NAT mode", you can modify the NAT gateway, then click "OK".

### 4. Configure Hostname and Static IP Address
#### 4.1 Configure Hostname
```bash
vim /etc/hostname
```
Modify the content of the file to your desired hostname, then save and exit.
#### 4.2 Configure Static IP Address
```bash
vim /etc/sysconfig/network-scripts/ifcfg-ens33
```
Modify `BOOTPROTO` to `static`, set `IPADDR` to your desired IP address, add `NETMASK` and set it to `255.255.255.0`, add `GATEWAY` and set it to the NAT gateway from VMware's Virtual Network Editor, add `DNS1` and set it to your desired DNS server (e.g., Google's `8.8.8.8`), keep other settings unchanged, then save and exit.
* Note: The IP addresses of `IPADDR` and `GATEWAY` must be within the subnet of the NAT gateway's IP address in VMware's Virtual Network Editor, and they must not conflict with IP addresses in the Windows host's subnet.

## FAQ
### Why Choose NAT for Network Type
NAT stands for Network Address Translation, a technology that converts private IP addresses to public IP addresses, enabling communication between different networks. With the depletion of IPv4 addresses, NAT effectively solves the problem of insufficient IP addresses and provides users with more stable network connections.
In the context of virtual machines, NAT can translate the IP addresses of Windows and virtual machines to be on the same subnet, enabling Windows access and communication between virtual machines without conflicting with IP addresses in the Windows host's subnet or affecting the Windows network.

### How to Install and Use the Vim Editor
Vim is a powerful text editor that can be downloaded using the following command:
```bash
yum install vim
```
In command mode, press `i` to enter insert mode, press `esc` to exit insert mode, and type `:wq` to save and exit the vim editor.

## Summary
This article mainly introduced the process of setting up virtual machines required for a Hadoop 3.x cluster, hoping it will be helpful to you.
