---
layout:     post
title:      "如何部署hadoop集群 (二)"
subtitle:   "让我们简单部署hadoop3.x集群中的HDK和Hadoop依赖"
date:       2025-3-2
author:     "Byolio"
header-img: "img/InstallJdkHadoop.jpg"
catalog: true
tags:
    - linux
    - hadoop
---
> 本文主要介绍了如何布置hadoop3.x集群的jdk和hadoop依赖

## 前言
我在[如何部署hadoop集群 (一)](https://byolio.top/2025/02/23/hadoopdeploy/)中已经布置好了一台虚拟机, 接下来我们需要在这一台虚拟机上安装jdk和hadoop依赖, 以便我们后续的hadoop集群部署。

## 什么是JDK
`JDK`是`Java Development Kit`的缩写，是Java开发工具包的缩写。它是Java开发人员使用的软件开发工具包，包含了Java运行环境（JRE）、Java编译器（javac）和Java调试器（jdb）等工具。JDK是Java开发的基础，它提供了Java程序开发所需的所有工具和环境。Hadoop3.x是基于Java开发的，因此需要安装JDK才能部署Hadoop3.x集群。

## Hadoop3.x的JDK依赖版本
我建议部署Hadoop3.x中, 部署JDK1.8(Java 8)作为其依赖版本, 许多Hadoop相关的工具和库都已经为JDK1.8优化，因此Java 8提供了最好的兼容性和性能。而Java 9+可能会导致一些问题, 因为Java 9+引入了模块化系统（Jigsaw），这会导致一些与Hadoop3.x相关的依赖或库出现兼容性问题。如果你已经部署Java 9+版本, 我将在FAQ中给出解决方案。


## 如何部署Hadoop与JDK依赖
### 1. 下载JDK1.8
点开[官网](https://www.oracle.com/hk/java/technologies/javase/javase8-archive-downloads.html), 选择`Linux x64`的tar.gz文件, 然后点击下载, 下载完成后, 将文件从windows上传到虚拟机(FAQ提供上传方案), 使用以下命令解压文件:
```bash
tar -zxvf jdk-8u301-linux-x64.tar.gz -C /dest/
```
注 : /dest/为你想要解压到的目录, 需要替换为你想要解压到的目录, 如/opt/等等
### 2. 下载hadoop3.x
打开[hadoop官网](https://hadoop.apache.org/releases.html), 选择版本并点击`Binary download`下的`binary`, 然后点击Http下的下载地址, 即可下载hadoop3.x的tar.gz文件, 下载完成后, 将文件从windows上传到虚拟机(FAQ提供上传方案), 使用以下命令解压文件:
```bash
tar -zxvf hadoop-3.3.3.tar.gz -C /dest/
```
注 : /dest/为你想要解压到的目录, 需要替换为你想要解压到的目录, 如/opt/等等
### 3. 配置启动JDK
为了清晰地组织不同应用的环境变量，避免/etc/profile变得混乱。我们可以创建一个新的文件，例如`/etc/profile.d/env.sh`
```bash
vim /etc/profile.d/env.sh
```
在文件中添加以下内容:
```bash
#JAVA_HOME
export JAVA_HOME=/opt/jdkfilename
export PATH=$PATH:$JAVA_HOME/bin
```
注 : /opt/jdkfilename为你解压JDK的目录, 需要替换为你解压JDK的目录(前面的/dest/), 如/opt/jdk-8u202-linux-x64
启动jdk:
```bash
source /etc/profile
```
输如`java -version`, 如果返回java版本号, 则说明安装成功
### 4. 配置启动hadoop
在上面的步骤中, 我们已经解压了hadoop3.x的tar.gz文件, 现在我们需要配置hadoop的环境变量, 打开`/etc/profile.d/env.sh`文件, 在文件中添加以下内容:
```bash
#HADOOP_HOME
export HADOOP_HOME=/opt/hadoopfilename
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```
注 : /opt/hadoopfilename为你解压hadoop3.x的目录, 需要替换为你解压hadoop3.x的目录(前面的/dest/), 如/opt/hadoop-3.4.0
启动hadoop:
```bash
source /etc/profile
```
输如`hadoop version`, 如果返回hadoop版本号, 则说明安装成功

## FAQ
### 打不开官网怎么办
请开启魔法, 并重新打开官网即可

### 如何将文件上传到虚拟机中
可以将文件从windows中上传到虚拟机中的软件有很多, 如xftp, tabby, winscp等等, 下载配置地址, 用户名, 密码后即可上传文件

### 如何解决Java 9+版本部署Hadoop3.x的问题
如果你已经部署Java 9+版本, 你可以按照以下步骤解决部署hadoop3.x集群的问题: \
先输入以下命令:
```bash
vim /opt/hadoopfilename/etc/hadoop/hadoop-env.sh
```
注: /opt/hadoopfilename为你解压hadoop3.x的目录, 需要替换为你解压hadoop3.x的目录(前面的/dest/), 如/opt/hadoop-3.4.0 \
然后在文件中的:
```bash
# Extra Java runtime options for all Hadoop commands. We don't support
# IPv6 yet/still, so by default the preference is set to IPv4.
# export HADOOP_OPTS="-Djava.net.preferIPv4Stack=true"
# For Kerberos debugging, an extended option set logs more information
# export HADOOP_OPTS="-Djava.net.preferIPv4Stack=true -Dsun.security.krb5.debug=true -Dsun.security.spnego.debug"
```
下面添加`export HADOOP_OPTS="$HADOOP_OPTS --add-opens java.base/java.lang=ALL-UNNAMED"`即可解决问题

## 总结
本文主要介绍了如何布置hadoop3.x集群的hadoop和jdk依赖, 以便我们后续的hadoop集群部署。