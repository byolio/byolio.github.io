---
layout: post
title: How to Deploy a Hadoop Cluster (Part 3)
subtitle: 让我们克隆vmware虚拟机并对其进行配置
date: 2025-3-12
author: Byolio
header-img: img/hadoopCopy.jpg
catalog: true
tags:
  - linux
  - hadoop
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2025-3-12-hadoopCopy/) | [English](https://www.byolio.blog/en/en/_posts/2025-3-12-hadoopCopy/)

> This article mainly introduces how to clone a VMware virtual machine and configure the cloned virtual machine

## Preface
In [How to Deploy a Hadoop Cluster (Part 2)](https://byolio.top/2025/03/02/InstallJdkHadoop/), I have already configured the JDK dependencies and installed Hadoop on one virtual machine. Next, we need to clone enough VMware virtual machines to simulate the scenario of multiple servers in a cluster.

## How Many Servers Should We Set Up
Although I recommend setting up 7 servers when building a Hadoop cluster:
* 3 servers for the Hadoop DataNode cluster
* 1 server for the Hadoop cluster's NameNode
* 1 server for the Hadoop cluster's SecondaryNameNode
* 1 server for the Hadoop cluster's ResourceManager
* 1 server for the Hadoop cluster's JobHistory
This can better simulate a production cluster scenario.
However, due to limited personal computer performance, when learning Hadoop clusters, we can set up 3 servers as follows:
* One server for the Hadoop cluster's DataNode, NameNode, and JobHistory
* One server for the Hadoop cluster's DataNode and SecondaryNameNode
* One server for the Hadoop cluster's DataNode and ResourceManager
This maximizes the purpose of learning Hadoop clusters.

Note: Be sure to place the NameNode, SecondaryNameNode, and ResourceManager nodes on different servers to better simulate a production cluster scenario.

## How to Properly Clone a Virtual Machine (This step requires shutting down the virtual machine to be cloned first)
We can use VMware's cloning feature to clone virtual machines, which can speed up the cluster setup.
Click on the virtual machine, right-click and select Manage -> Clone to enter the cloning interface. In the cloning method, choose Full Clone, then click Next, select the name and location for the cloned virtual machine, and then click Finish.

Then open the settings of the newly cloned virtual machine -> Network Adapter -> Advanced, and click Generate next to the MAC address (MAC addresses must be different and unique on the same network to prevent network connection issues), then click OK.

## How to Configure the Virtual Machine
If you have cloned at least three virtual machines, then we need to open the newly created virtual machines and configure them.

* Note: Do not open more than one newly created virtual machine at the same time, otherwise network connection issues may occur. Please complete the configuration on one virtual machine before opening another for configuration.
### Configure Hostname and Static IP
#### Configure Hostname
```bash
vim /etc/hostname
```
Modify the content in the file to the hostname you want (must be different from other hosts), then save and exit.
#### Configure Static IP
```bash
vim /etc/sysconfig/network-scripts/ifcfg-ens33
```
Modify the IPADDR to the IP address you want (must be on the same subnet as other Hadoop hosts), then save and exit.\\

After modifying the hostname and IP address of all hosts, the configuration for this step is complete.

### Configure the hosts File
To facilitate subsequent cluster configuration and avoid the need for extensive configuration modifications after changing virtual machine IPs, we need to configure the hosts file. This allows us to use hostnames to access other hosts in the cluster.
#### Configure the hosts file on Linux
```bash
vim /etc/hosts
```
Add the hostname and IP address at the end in the following format: ip hostname \\
For example:
```bash
192.168.138.128 hadoop101    # ip hostname
192.168.138.129 hadoop103
192.168.138.130 hadoop104
```
* Note: Replace the IP addresses and hostnames with those of your Hadoop cluster, and ensure that each IP address corresponds to exactly one hostname.

After configuring all hosts as above, the Linux hosts file configuration is complete.

#### Configure the hosts file on Windows
Open the `hosts` file located at `C:\Windows\System32\drivers\etc\`, and add the hostname and IP address at the end in the following format: ip hostname \\
For example:
```txt
192.168.138.128 hadoop101
192.168.138.129 hadoop103
192.168.138.130 hadoop104
```
* Note: Replace the IP addresses and hostnames with those of your Hadoop cluster, and ensure that each IP address corresponds to exactly one hostname.
#### Verify Configuration
On a Hadoop host, use the `ping 主机名` command to test whether the configuration is successful. For example:
```bash
ping hadoop103
```
If the ping succeeds, the configuration is successful.

### Configure SSH Passwordless Login
In Hadoop, when we use one server to control other servers via SSH, we need to enter a password. Therefore, we need to configure SSH passwordless login to save a lot of time entering passwords.
The code is as follows:
```bash
ssh-keygen -t rsa
```
This generates a key pair. Press Enter all the way. The generated keys are in the `~/.ssh/` directory, where `id_rsa` is the private key and `id_rsa.pub` is the public key. \\
Then use
```bash
ssh-copy-id hostname
```
to copy the public key to other hosts, where `hostname` is the hostname of the other host. For example:
```bash
ssh-copy-id hadoop103
```
* Note:
    1. When copying the public key, you will be prompted to enter the password of the other host. After entering it, the copy will succeed.
    2. When copying the public key, you need to copy it to all hosts, including yourself, otherwise you may not be able to log in.

After configuring SSH passwordless login for each host, the SSH passwordless login configuration for the Hadoop hosts is complete.

#### 
## FAQ
### Why Shut Down Before Cloning
Because when cloning a virtual machine, all files of the virtual machine are copied, including some cache and temporary files. These files are automatically cleared after shutdown. If you clone without shutting down, these files may cause the cloned virtual machine to fail to start properly.
### What is the Use of the hosts File
When performing domain name resolution, operating systems (such as Windows, Linux, macOS, etc.) first check the entries in the hosts file. If there is a matching entry in the file, the system will directly use that IP address without querying an external DNS server.
### Why Copying the Public Key to Other Hosts Enables Passwordless Login
When using SSH for remote login, the client decrypts the encrypted data sent by the server using its private key, proving that it possesses the private key corresponding to the public key in the authorized_keys file, thereby completing authentication and allowing login. The copied public key is added to the authorized_keys file, so in subsequent login processes, the client will automatically use that public key for verification.
