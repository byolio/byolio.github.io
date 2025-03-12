---
layout:     post
title:      "如何部署hadoop集群 (三)"
subtitle:   "让我们克隆vmware虚拟机并对其进行配置"
date:       2025-3-12
author:     "Byolio"
header-img: "img/hadoopCopy.jpg"
catalog: true
tags:
    - linux
    - hadoop
---
> 本文主要介绍了如何克隆vmware虚拟机, 并对克隆后的虚拟机其进行配置

## 前言
我在[如何部署hadoop集群 (二)](https://byolio.top/2025/03/02/InstallJdkHadoop/)中已经配置好一台虚拟机的jdk依赖并安装了hadoop, 接下来我们需要克隆足够的vmware虚拟机模拟集群中大量服务器的场景

## 应该搭建多少服务器
虽然我建议在搭建hadoop集群时, 搭建7台服务器:
* 3台服务器用于搭建hadoop的DataNode集群
* 1台服务器用于搭建hadoop集群的NameNode节点
* 1台服务器用于搭建hadoop集群的SecondaryNameNode节点
* 1台服务器用于搭建hadoop集群的ResourceManager节点
* 1台服务器用于搭建hadoop集群的JobHistory节点
这样可以更好地模拟生产集群的场景。
但介于个人电脑的性能有限, 在学习hadoop集群时, 我们可以这样搭建3台服务器:
* 一台服务器用于搭建hadoop集群的DataNode和NameNode和JobHistory节点
* 一台服务器用于搭建hadoop集群的DataNode和SecondaryNameNode节点
* 一台服务器用于搭建hadoop集群的DataNode和ResourceManager节点
用于最大程度的实现学习hadoop集群的目的。

注 : 一定要将NameNode, SecondaryNameNode, ResourceManager节点分开放在不同服务器上, 这样可以更好地模拟生产集群的场景。

## 如何正确克隆虚拟机 (本步骤需要先关闭要被克隆的虚拟机)
我们可以使用vmware的克隆功能来克隆虚拟机, 这样可以更加快速完成集群的搭建。
点击虚拟机, 右键选择 管理->克隆 进入克隆界面, 在克隆方式中选择完全克隆, 然后点击下一步, 选择克隆虚拟机的名称和位置, 然后点击完成即可。

然后点开新克隆的设置界面->网络适配器->高级, 然后在MAC地址旁边点击生成(同一网络下MAC地址必须不一样且唯一以防网路链接发生问题), 然后点击确定即可。

## 如何配置虚拟机
如果你的虚拟机已经克隆好至少三台, 那么接下来我们需要打开新建的虚拟机, 并对其进行配置。

* 注意 : 请不要同时打开一台以上刚建的虚拟机, 否则会出现网络链接问题, 请在一台虚拟机上完成配置后, 再打开另一台虚拟机进行配置。
### 配置主机名和静态地址
#### 配置主机名
```bash
vim /etc/hostname
```
将文件中的内容修改为你想要的主机名(要求和其他主机不一样), 然后保存并退出
#### 配置静态地址
```bash
vim /etc/sysconfig/network-scripts/ifcfg-ens33
```
将其中的IPADDR修改为你想要的IP地址(需要与其他hadoop主机在同一网段下), 然后保存并退出\

将所有主机的主机名和IP地址修改完毕后, 完成该步骤的配置。

### 配置hosts文件
为了方便接下来的集群配置和防止出现虚拟机ip修改后需要进行大量配置修改的情况, 我们需要配置hosts文件, 这样可以方便我们在集群中使用主机名来访问集群中的其他主机。
#### 配置linux的hosts文件
```bash
vim /etc/hosts
```
在其中按照如下格式在末尾添加主机名和IP地址: ip 主机名 \
如:
```bash
192.168.138.128 hadoop101    # ip 主机名
192.168.138.129 hadoop103
192.168.138.130 hadoop104
```
* 注 : 将ip地址和主机名替换为hadoop的ip地址和主机名, 并确保ip地址和主机名一一对应。

将所用主机经过上述配置后, 完成linux的hosts文件配置。

#### 配置windows的hosts文件
打开`C:\Windows\System32\drivers\etc\`路径下的`hosts`文件, 在其中按照如下格式在末尾添加主机名和IP地址:   ip 主机名 \
如:
```txt
192.168.138.128 hadoop101
192.168.138.129 hadoop103
192.168.138.130 hadoop104
```
* 注 : 将ip地址和主机名替换为hadoop的ip地址和主机名, 并确保ip地址和主机名一一对应。
### 判断是否配置成功
在hadoop主机上使用`ping 主机名`命令测试是否配置成功, 如:
```bash
ping hadoop103
```
如果能ping通, 则说明配置成功。

### 配置ssh免密登录
在hadoop中我们使用一台服务器通过ssh操控其他服务器需要输入密码, 因此我们需要配置ssh免密登录, 节省掉大量输入密码的时间
代码如下:
```bash
ssh-keygen -t rsa
```
用于生成密钥, 一路回车即可, 生成的密钥在`~/.ssh/`目录下, 其中`id_rsa`是私钥, `id_rsa.pub`是公钥。 \
然后使用
```bash
ssh-copy-id hostname
```
将公钥复制到其他主机上, 其中`hostname`是其他主机的主机名, 如:
```bash
ssh-copy-id hadoop103
```
* 注 : 
    1. 复制公钥时, 会提示你输入其他主机的密码, 输入后即可复制成功。
    2. 复制公钥时需要将公钥需要通过指令复制给包括自己在内的全部主机, 否则可能会出现无法登录的情况。

配置完成每一个主机的ssh免密登录后, 完成hadoop主机的ssh免密登录配置。

#### 
## FAQ
### 为什么要先关机再克隆
因为克隆虚拟机时, 会复制虚拟机的所有文件, 包括部分缓存和临时文件, 这些文件在关机后会被自动清除掉, 如果不关机直接克隆, 这些文件可能会导致克隆后的虚拟机无法正常启动。
### hosts文件有什么用
操作系统（如 Windows、Linux、macOS 等）在进行域名解析时，首先会检查hosts文件中的条目。如果文件中有匹配的条目，系统会直接使用该IP地址，而不需要查询外部的DNS服务器。
### 为什么将公钥给其他的主机可以免密登录
在使用SSH进行远程登录时，客户端通过其私钥解密服务器发送的加密数据，以证明它拥有与authorized_keys中公钥匹配的私钥，从而完成身份认证并允许登录。复制的公钥会被添加到authorsized_keys文件中，因此在后续的登录过程中，客户端会自动使用该公钥进行验证。