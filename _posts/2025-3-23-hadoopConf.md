---
layout:     post
title:      "如何部署hadoop集群 (五)"
subtitle:   "让我们完成hadoop集群配置文件的基础配置"
date:       2025-3-23
author:     "Byolio"
header-img: "img/hadoopConf.jpg"
catalog: true
tags:
    - linux
    - hadoop
---
> 本文主要介绍hadoop部署过程中的相关配置文件的基础配置, 

## 引言
我在[如何部署hadoop集群 (四)](https://byolio.top/2025/03/16/hadoopExtend/)中已经讲述了部分配置时的注意事项, 接下来我们需要配置配置文件, 用于后续hadoop集群的启动和功能的实现

## 配置hdfs
`HDFS`(Hadoop Distributed File System)是`Apache Hadoop`生态系统的核心组件之一，是一个专为大数据存储和处理而设计的分布式文件系统。
### 打开配置文件

```bash
cd /path/to/hadoopfile/ # 进入hadoop安装目录
cd etc/hadoop/ # 进入hadoop配置文件目录
cd hdfs-site.xml # 进入hdfs配置文件
```
* 注 : /path/to/hadoopfile/为hadoop安装目录, 根据需要进行修改
### 配置文件内容

```xml
<configuration>
<property>
  <name>dfs.namenode.http-address</name>
  <value>hadoopname01:9870</value>
</property>
<property>
  <name>dfs.namenode.secondary.http-address</name>
  <value>hadoopname02:9868</value>
</property>
</configuration>
```
* 注 : hadoopname01为namenode的主机名, 9870为namenode的端口号, hadoopname02为secondarynamenode的主机名, 9868为secondarynamenode的端口号, 根据需要进行修改, hadoopname01和hadoopname02建议使用不同的主机名, 以实现hadoop集群的高可用性
### 配置内容说明
1. dfs.namenode.http-address - hadoopname01:9870
这个属性指定了HDFS NameNode的HTTP地址，用于通过Web UI界面访问NameNode。hadoopname01:port01表示NameNode的主机地址和端口号。通过这个地址，用户可以访问HDFS的Web UI（通常是NameNode的管理页面），进行文件系统的管理、查看文件目录结构、查看文件块分布等操作。
2. dfs.namenode.secondary.http-address - hadoopname02:9868
这个属性指定了HDFS Secondary NameNode的HTTP地址，用于通过Web UI界面访问Secondary NameNode。hadoopname02:6868表示Secondary NameNode的主机地址和端口号。Secondary NameNode的主要功能是定期下载HDFS的元数据（fsimage和edits日志），进行合并和压缩，并将结果上传回NameNode。通过这个地址，用户可以访问Secondary NameNode的Web管理界面，查看其运行状态等。

## 配置yarn
`YARN`(Yet Another Resource Negotiator)是`Apache Hadoop`生态系统的另一个核心组件，它是一个资源管理和调度系统，用于协调和管理集群中的计算资源。
### 打开配置文件
```bash
cd /path/to/hadoopfile/ # 进入hadoop安装目录
cd etc/hadoop/ # 进入hadoop配置文件目录
cd yarn-site.xml # 进入yarn配置文件
```
* 注 : /path/to/hadoopfile/为hadoop安装目录, 根据需要进行修改
### 配置文件内容

```xml
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoopname03</value>
  </property>
  <property>
    <name>yarn.nodemanager.env-whitelist</name>
    <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
  </property>
<property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
  </property>

  <property>
    <name>yarn.log.server.url</name>
    <value>http://hadoopname01:19888/jobhistory/logs</value>
  </property>

  <property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>604800</value>
  </property>
</configuration>
```
* 注 : hadoopname03为resourcemanager的主机名, 根据需要进行修改, hadoopname02为logserver的主机名, 根据需要进行修改
### 配置内容说明
1. yarn.nodemanager.aux-services - mapreduce_shuffle 这个属性定义了YARN NodeManager上启用的辅助服务。这里的mapreduce_shuffle指定了MapReduce作业的Shuffle过程将由NodeManager处理。Shuffle是MapReduce作业的一个关键步骤，负责在Map阶段与Reduce阶段之间传输数据。启用mapreduce_shuffle服务意味着NodeManager将为MapReduce作业提供这个功能。
2. yarn.resourcemanager.hostname - hadoopname03 该属性指定了YARN ResourceManager的主机名或地址。hadoopname03是YARN资源管理器的主机名，负责管理集群中的计算资源，调度作业，并协调任务的执行。
3. yarn.nodemanager.env-whitelist - JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME 这个属性设置了NodeManager环境变量的白名单。当NodeManager启动时，它会从白名单中读取并允许这些环境变量。这些变量通常是指向Hadoop和Java相关的环境配置，如JAVA_HOME、HADOOP_COMMON_HOME等。此设置确保NodeManager能够访问正确的环境变量，以便能够正确运行和管理任务。
4. yarn.log-aggregation-enable - true 这个属性启用YARN的日志聚合功能。当设置为true时，YARN会将所有执行任务的日志文件聚合到一个集中位置，而不是保存在单独的节点上。这有助于简化日志管理，特别是在处理大规模集群时。
5. yarn.log.server.url - `http://hadoopname01:19888/jobhistory/logs` 该属性指定了日志服务器的URL地址。`http://hadoopname01:19888/jobhistory/logs` 是YARN日志服务器的Web界面地址，可以通过此地址查看集群中作业的日志。用户可以访问这个地址来查看作业的执行日志、错误信息以及详细的运行状态。
6. yarn.log-aggregation.retain-seconds - 604800 这个属性设置了日志聚合后保留日志的时长（以秒为单位）。604800秒相当于7天。这意味着YARN将保留聚合后的作业日志7天，超过此时间的日志会被自动删除。此设置帮助管理员管理存储空间，防止日志文件无限制地占用磁盘空间。

## 配置mapreduce
`MapReduce`是`Apache Hadoop`生态系统的核心计算模型，它是一个用于大规模数据集处理的编程模型和实现框架。MapReduce的设计灵感来源于函数式编程，它将复杂的分布式计算问题分解为两个主要阶段：Map和Reduce。
### 打开配置文件
```bash
cd /path/to/hadoopfile/ # 进入hadoop安装目录
cd etc/hadoop/ # 进入hadoop配置文件目录
cd mapred-site.xml # 进入mapreduce配置文件
```
* 注 : /path/to/hadoopfile/为hadoop安装目录, 根据需要进行修改
### 配置文件内容

```xml
<configuration>
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
</property>
<property>
  <name>mapreduce.jobhistory.address</name>
  <value>hadoopname01:10020</value>
</property>
<property>
  <name>mapreduce.jobhistory.webapp.address</name>
  <value>hadoopname01:19888</value>
</property>
</configuration>
```
* 注 : hadoopname01为jobhistory的主机名, 根据需要进行修改, hadoopname01为logserver的主机名, 根据需要进行修改
### 配置内容说明
1. mapreduce.framework.name - yarn 这个属性指定了MapReduce作业的执行框架。这里设置为yarn，意味着MapReduce作业将由YARN（Yet Another Resource Negotiator）框架来调度和执行。YARN是Hadoop的资源管理器，用于动态地分配计算资源给不同的作业。
2. mapreduce.jobhistory.address - hadoopname01:10020 该属性指定了MapReduce作业历史服务器的地址。hadoopname01:10020是保存作业历史信息的服务器地址和端口。当MapReduce作业执行完成后，相关的作业历史信息会被记录在这个服务器上，供后续查询和分析使用。
3. mapreduce.jobhistory.webapp.address - hadoopname01:19888 这个配置定义了MapReduce作业历史服务器的Web界面的访问地址。hadoopname01:19888是作业历史Web应用程序的URL地址，可以通过浏览器访问此地址来查看和管理已完成的MapReduce作业的历史数据，如作业状态、执行时间、任务统计等信息。

## 配置core-site
`core-site.xml`是`Apache Hadoop`生态系统中的一个核心配置文件，它包含了Hadoop集群的全局配置信息。这个文件定义了Hadoop的核心属性，例如Hadoop的名称、数据存储路径、文件系统实现类等。
### 打开配置文件
```bash
cd /path/to/hadoopfile/ # 进入hadoop安装目录
cd etc/hadoop/ # 进入hadoop配置文件目录
cd core-site.xml # 进入core配置文件
```
* 注 : /path/to/hadoopfile/为hadoop安装目录, 根据需要进行修改
### 配置文件内容
```xml
<configuration>
<property>
  <name>fs.defaultFS</name>
  <value>hdfs://hadoopname01:8020</value>
</property>
<property>
  <name>hadoop.tmp.dir</name>
  <value>/path/to/data</value>
</property>
<property>
  <name>hadoop.http.staticuser.user</name>
  <value>byolio</value>
</property>
</configuration>
```
* 注 : hadoopname01为namenode的主机名, 根据需要进行修改, /path/to/data为数据存储路径, 根据需要进行修改, byolio为hadoop的用户名, 根据需要进行修改
### 配置内容说明
1. fs.defaultFS - hdfs://hadoopname01:8020
这是Hadoop分布式文件系统(HDFS)的默认文件系统的内部URI。它指定了HDFS的NameNode服务器地址和端口(hadoopname01:8020)。所有文件操作如果没有明确指定文件系统，都会默认使用这个地址。
2. hadoop.tmp.dir - /path/to/data
这个属性设置Hadoop的临时目录路径。Hadoop会在这个目录下存储各种临时文件和中间数据。这包括NameNode和DataNode的本地存储位置、MapReduce作业的临时文件等。建议将此目录设置在有足够空间的磁盘上。
3. hadoop.http.staticuser.user - byolio
这个配置定义了Hadoop Web UI(如HDFS NameNode Web UI、ResourceManager Web UI等)的静态用户名。当通过Web界面访问HDFS文件时，会使用这个用户身份。这对于代理访问和权限控制很重要。

## FAQ
### 为什么要配置历史服务器功能
当MapReduce作业运行完成后，ResourceManager不再保存这些作业的详细信息和日志。需要配置历史服务器保存这些信息，使管理员和开发人员能够查看已完成作业的执行细节, 用于排查错误。

### 为什么要配置日志聚集功能
在没有启用日志聚集的情况下，每个节点的日志都存储在本地，这会导致在查询、访问和分析日志时变得复杂，尤其是当集群规模很大时。启用日志聚集后，所有作业的日志会集中存储在一个位置，简化了日志的查找和管理。

### 为什么无法在Hadoop Web UI上进行删除等操作
如果没有配置静态用户名, Hadoop Web UI则会使用默认的dr.who用户, 其没有权限去在Hadoop Web UI上进行删除等操作, 因此需要配置静态用户名配置为有权限进行删除等操作的用户名, 以便于管理员和开发人员能够在Hadoop Web UI上进行操作。

### 如何查找默认配置文件并选择性配置文件
阅读官方文档, 并查看源码从中找出:
1. core-default.xml : 包含了Hadoop的核心配置默认信息, 例如Hadoop的名称、数据存储路径、文件系统实现类等
2. hdfs-default.xml : 包含了Hadoop分布式文件系统(HDFS)的默认配置信息, 例如HDFS的NameNode服务器地址和端口、DataNode的本地存储位置、MapReduce作业的临时文件等
3. yarn-default.xml : 包含了Hadoop的资源管理和调度系统(YARN)的默认配置信息, 例如YARN的资源管理器地址和端口、MapReduce作业的执行模式、日志聚集功能等
4. mapred-default.xml : 包含了Hadoop的MapReduce框架的默认配置信息, 例如MapReduce作业的执行模式、日志聚集功能、历史服务器功能等
并根据其默认配置文件选择性配置core-site.xml、hdfs-site.xml、yarn-site.xml、mapred-site.xml中的配置信息, 以实现自定义的Hadoop集群配置。

### 那些是常用配置端口
分布式系统中涉及大量的服务和进程。如果没有预先定义好端口，可能会导致不同服务或进程使用相同的端口，从而引发端口冲突，导致服务不可用或系统不稳定。因此大家都使用一些hadoop3.x常用的默认端口:
NameNode 内部通信端口 : 8020/9000/9820
hadoop web ui 通信端口 : 9870
mapreduce web ui 端口 : 8088
历史服务器通信端口 : 19888

## 总结
本文主要介绍了hadoop集群配置文件的基础配置, 包括hdfs、yarn、mapreduce、core等配置文件的配置, 并给出了配置文件的配置内容和配置内容的说明, 方便你更好地理解hadoop集群的配置文件, 并能够更好地配置hadoop集群, 实现hadoop集群的高可用性和高性能。
