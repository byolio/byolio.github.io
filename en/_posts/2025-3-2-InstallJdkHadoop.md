---
layout: post
title: How to Deploy a Hadoop Cluster (Part 2)
subtitle: 让我们简单部署hadoop3.x集群中的HDK和Hadoop依赖
date: 2025-3-2
author: Byolio
header-img: img/InstallJdkHadoop.jpg
catalog: true
tags:
  - linux
  - hadoop
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2025-3-2-InstallJdkHadoop/) | [English](https://www.byolio.blog/en/en/_posts/2025-3-2-InstallJdkHadoop/)

> This article mainly introduces how to set up the JDK and Hadoop dependencies for a Hadoop 3.x cluster.

## Preface
In [How to Deploy a Hadoop Cluster (Part 1)](https://byolio.top/2025/02/23/hadoopdeploy/), I have already set up a virtual machine. Next, we need to install JDK and Hadoop dependencies on this virtual machine for subsequent Hadoop cluster deployment.

## What is JDK
`JDK` stands for `Java Development Kit`, which is a Java development toolkit. It is a software development kit used by Java developers, including the Java Runtime Environment (JRE), Java compiler (javac), and Java debugger (jdb). JDK is the foundation of Java development, providing all the tools and environment needed for Java program development. Hadoop 3.x is developed based on Java, so JDK must be installed to deploy a Hadoop 3.x cluster.

## JDK Dependency Version for Hadoop 3.x
I recommend deploying JDK 1.8 (Java 8) as the dependency version for Hadoop 3.x. Many Hadoop-related tools and libraries have been optimized for JDK 1.8, so Java 8 provides the best compatibility and performance. Java 9+ may cause some issues because Java 9+ introduced the modular system (Jigsaw), which can lead to compatibility problems with some dependencies or libraries related to Hadoop 3.x. If you have already deployed Java 9+, I will provide a solution in the FAQ.

## How to Deploy Hadoop and JDK Dependencies
### 1. Download JDK 1.8
Open the [official website](https://www.oracle.com/hk/java/technologies/javase/javase8-archive-downloads.html), select the `Linux x64` tar.gz file, and click download. After downloading, upload the file from Windows to the virtual machine (FAQ provides upload methods), and use the following command to extract the file:
```bash
tar -zxvf jdk-8u301-linux-x64.tar.gz -C /dest/
```
Note: /dest/ is the directory where you want to extract the file; replace it with your desired directory, such as /opt/, etc.
### 2. Download Hadoop 3.x
Open the [Hadoop official website](https://hadoop.apache.org/releases.html), select the version, click `binary` under `Binary download`, then click the download link under HTTP to download the Hadoop 3.x tar.gz file. After downloading, upload the file from Windows to the virtual machine (FAQ provides upload methods), and use the following command to extract the file:
```bash
tar -zxvf hadoop-3.3.3.tar.gz -C /dest/
```
Note: /dest/ is the directory where you want to extract the file; replace it with your desired directory, such as /opt/, etc.
### 3. Configure and Start JDK
To clearly organize environment variables for different applications and avoid cluttering /etc/profile, we can create a new file, for example, `/etc/profile.d/env.sh`:
```bash
vim /etc/profile.d/env.sh
```
Add the following content to the file:
```bash
#JAVA_HOME
export JAVA_HOME=/opt/jdkfilename
export PATH=$PATH:$JAVA_HOME/bin
```
Note: /opt/jdkfilename is the directory where you extracted JDK; replace it with your JDK extraction directory (the previous /dest/), such as /opt/jdk-8u202-linux-x64.
Start JDK:
```bash
source /etc/profile
```
Type `java -version`. If the Java version number is returned, the installation is successful.
### 4. Configure and Start Hadoop
In the previous steps, we have already extracted the Hadoop 3.x tar.gz file. Now we need to configure Hadoop's environment variables. Open the `/etc/profile.d/env.sh` file and add the following content:
```bash
#HADOOP_HOME
export HADOOP_HOME=/opt/hadoopfilename
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```
Note: /opt/hadoopfilename is the directory where you extracted Hadoop 3.x; replace it with your Hadoop 3.x extraction directory (the previous /dest/), such as /opt/hadoop-3.4.0.
Start Hadoop:
```bash
source /etc/profile
```
Type `hadoop version`. If the Hadoop version number is returned, the installation is successful.

## FAQ
### What if I cannot open the official website?
Please enable a VPN and reopen the official website.

### How to upload files to the virtual machine?
There are many software tools that can upload files from Windows to the virtual machine, such as xftp, Tabby, WinSCP, etc. After configuring the address, username, and password, you can upload files.

### How to solve the problem of deploying Hadoop 3.x with Java 9+?
If you have already deployed Java 9+, you can follow these steps to solve the problem of deploying a Hadoop 3.x cluster: \
First, enter the following command:
```bash
vim /opt/hadoopfilename/etc/hadoop/hadoop-env.sh
```
Note: /opt/hadoopfilename is the directory where you extracted Hadoop 3.x; replace it with your Hadoop 3.x extraction directory (the previous /dest/), such as /opt/hadoop-3.4.0. \
Then, in the file, add the following line after:
```bash
# Extra Java runtime options for all Hadoop commands. We don't support
# IPv6 yet/still, so by default the preference is set to IPv4.
# export HADOOP_OPTS="-Djava.net.preferIPv4Stack=true"
# For Kerberos debugging, an extended option set logs more information
# export HADOOP_OPTS="-Djava.net.preferIPv4Stack=true -Dsun.security.krb5.debug=true -Dsun.security.spnego.debug"
```
Add `export HADOOP_OPTS="$HADOOP_OPTS --add-opens java.base/java.lang=ALL-UNNAMED"` to solve the problem.

## Summary
This article mainly introduces how to set up the Hadoop and JDK dependencies for a Hadoop 3.x cluster, in preparation for subsequent Hadoop cluster deployment.
