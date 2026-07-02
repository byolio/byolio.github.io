---
layout:     post
title:      "如何部署hadoop集群 (六)"
subtitle:   "让我们完成hadoop集群的初始化和启动与关闭和删除"
date:       2025-3-30
author:     "Byolio"
header-img: "img/hadoopStart.jpg"
catalog: true
tags:
    - linux
    - hadoop
---
> 本文主要介绍hadoop部署过程最后的初始化和启动与关闭步骤

## 引言
经过了前面的虚拟机部署和hadoop集群及其依赖安装与配置 接下来我们需要完成hadoop集群的最后一步--初始化和启动与关闭和删除

## 配置workers配置文件
先补充一个配置文件, 用于指定hadoop集群中的所有节点
```bash
cd /path/to/hadoopfile/etc/hadoop/
vim workers
```
* 注 : /path/to/hadoopfile/为hadoop安装目录, 根据需要进行修改
在其中添加所有节点的主机名, 每个节点一行, 如:
```bash
hadoopname01
hadoopname02
```

## 初始化hdfs
```bash
cd /path/to/hadoopfile/ # 进入hadoop安装目录
```
* 注 : /path/to/hadoopfile/为hadoop安装目录, 根据需要进行修改
在首次启动 HDFS 之前，需要格式化 NameNode：
```bash
hdfs namenode -format 
```
* 注 : 格式化 NameNode 应该在 NameNode 节点上执行，2NN(Secondary NameNode)节点不需要执行格式化

## 启动集群
### 启动hdfs
先切换到非root账户, 原因前面[文章](https://byolio.top/2025/03/16/hadoopExtend/)有说过
```bash
su - hadoopName # 切换到hadoop用户
```
* 注 : hadoopName为非root用户, 根据需要进行修改
```bash
start-dfs.sh # 启动hdfs
```
* 注 : 启动hdfs应该在NameNode节点上执行, 2NN(Secondary NameNode)等其他节点不需要执行启动

### 启动yarn
```bash
start-yarn.sh # 启动yarn
```
* 注 : 启动yarn应该在ResourceManager节点上执行, 其他节点不需要执行启动

### 启动historyserver
```bash
/path/to/hadoopfile/bin/mapred --daemon start historyserver
```
* 注 : /path/to/hadoopfile/为hadoop安装目录, 根据需要进行修改, 启动historyserver应该在JobHistoryServer节点上执行, 其他节点不需要执行启动
### 查看集群状态
```bash
jps
```
如果在对应节点上看到jps以外启动的进程, 则表示集群启动成功

## 关闭集群
### 关闭historyserver
```bash
/path/to/hadoopfile/bin/mapred --daemon stop historyserver
```
* 注 : /path/to/hadoopfile/为hadoop安装目录, 根据需要进行修改, 关闭historyserver应该在JobHistoryServer节点上执行, 其他节点不需要执行关闭
### 关闭yarn
```bash
stop-yarn.sh # 关闭yarn
```
* 注 : 关闭yarn应该在ResourceManager节点上执行, 其他节点不需要执行关闭
### 关闭hdfs
```bash
stop-dfs.sh 
```
* 注 : 关闭hdfs应该在NameNode节点上执行, 2NN(Secondary NameNode)等其他节点不需要执行关闭
## 查看集群状态
```bash
jps
```
如果没有看到jps以外的进程, 则表示集群关闭成功

## 删除集群
如果配置出现问题, 可以删除集群, 重新部署
```bash
rm -rf /path/to/data
```
* 注 : /path/to/data为hadoop数据目录, 根据需要进行修改, 删除hadoop数据目录
```bash
rm -rf /path/to/logs
```
* 注 : /path/to/logs为hadoop日志目录, 根据需要进行修改, 删除hadoop日志目录

然后重新初始格式化hdfs即可
* 注 : 每个DataNode节点都需要删除对应的hadoop数据目录, 和每个节点上的hadoop日志目录

## FAQ
### 为什么workers配置后启动集群会报错
1️⃣ 确保每个节点的主机名都正确配置在workers文件中同时, 也确保对应主机名和IP地址正确配置在/etc/hosts文件中。
2️⃣ workers文件中一行只能配置一个节点的主机名, 不能配置多个节点的主机名。
3️⃣ 不能有任何空格或注释行。
### 为什么要格式化NameNode
格式化NameNode的主要目的是初始化或重新初始化HDFS的元数据, 包括文件系统的目录结构、文件的块信息、权限等。
### 正确的启动和关闭顺序及其为什么
#### 启动顺序
1️⃣ 启动 NameNode（HDFS 元数据管理）
2️⃣ 启动 Secondary NameNode（合并元数据日志，可选）
3️⃣ 启动 DataNode（存储HDFS数据块）
4️⃣ 启动 ResourceManager（YARN任务调度）
5️⃣ 启动 NodeManager（执行计算任务的Worker节点）
6️⃣ 启动 JobHistory Server（存储已完成作业的历史记录）
#### 关闭顺序
1️⃣ 关闭 JobHistory Server（确保历史数据完整写入）
2️⃣ 关闭 NodeManager（防止正在运行的任务被强制中断）
3️⃣ 关闭 ResourceManager（防止YARN继续接受新任务）
4️⃣ 关闭 DataNode（避免 HDFS 误判数据丢失）
5️⃣ 关闭 Secondary NameNode（避免NameNode关闭后日志合并出错）
6️⃣ 关闭 NameNode（最后关闭，确保元数据完整）
#### 为什么要按照这个顺序
1️⃣ HDFS必须先启动，因为YARN和JobHistory Server依赖HDFS。
2️⃣ DataNode 必须在NameNode之后启动，否则会启动safe mode(如果有部分DataNode还没有响应，或者HDFS数据存在不一致性，NameNode就会进入Safe Mode，防止错误数据写入系统)。
3️⃣ JobHistory Server需要在YARN之后启动，否则作业历史无法保存。
4️⃣ 关闭时顺序相反，确保元数据完整，防止NameNode误判数据丢失。
### 为什么要删除data目录和logs目录
1️⃣ 旧的data目录中存储着之前的块数据，而格式化NameNode时会重新初始化元数据。如果旧数据依然存在，可能导致新的元数据与实际存储不匹配，从而引发错误或数据不一致问题。
2️⃣ 日志混乱：logs目录里的旧日志可能混淆新系统的调试信息，会给故障排查带来困扰