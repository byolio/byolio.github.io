---
layout: post
title: How to Deploy a Hadoop Cluster (Part 5)
subtitle: 让我们完成hadoop集群配置文件的基础配置
date: 2025-3-23
author: Byolio
header-img: img/hadoopConf.jpg
catalog: true
tags:
  - linux
  - hadoop
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2025-3-23-hadoopConf/) | [English](https://www.byolio.blog/en/en/_posts/2025-3-23-hadoopConf/)

> This article mainly introduces the basic configuration of relevant configuration files during the Hadoop deployment process.

## Introduction
In [How to Deploy a Hadoop Cluster (Part 4)](https://byolio.top/2025/03/16/hadoopExtend/), I have already discussed some precautions for configuration. Next, we need to configure the configuration files for the subsequent startup and functionality implementation of the Hadoop cluster.

## Configuring HDFS
`HDFS` (Hadoop Distributed File System) is one of the core components of the `Apache Hadoop` ecosystem. It is a distributed file system designed for big data storage and processing.
### Opening the Configuration File

```bash
cd /path/to/hadoopfile/ # Enter the Hadoop installation directory
cd etc/hadoop/ # Enter the Hadoop configuration file directory
cd hdfs-site.xml # Enter the HDFS configuration file
```
* Note: /path/to/hadoopfile/ is the Hadoop installation directory, modify it as needed.
### Configuration File Content

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
* Note: hadoopname01 is the hostname of the NameNode, 9870 is the port number of the NameNode, hadoopname02 is the hostname of the Secondary NameNode, 9868 is the port number of the Secondary NameNode. Modify them as needed. It is recommended to use different hostnames for hadoopname01 and hadoopname02 to achieve high availability of the Hadoop cluster.
### Configuration Content Explanation
1. dfs.namenode.http-address - hadoopname01:9870
This property specifies the HTTP address of the HDFS NameNode, used to access the NameNode via the Web UI interface. hadoopname01:port01 indicates the host address and port number of the NameNode. Through this address, users can access the HDFS Web UI (usually the NameNode management page) to manage the file system, view the file directory structure, view file block distribution, and perform other operations.
2. dfs.namenode.secondary.http-address - hadoopname02:9868
This property specifies the HTTP address of the HDFS Secondary NameNode, used to access the Secondary NameNode via the Web UI interface. hadoopname02:6868 indicates the host address and port number of the Secondary NameNode. The main function of the Secondary NameNode is to periodically download HDFS metadata (fsimage and edits logs), merge and compress them, and upload the results back to the NameNode. Through this address, users can access the Web management interface of the Secondary NameNode to view its running status, etc.

## Configuring YARN
`YARN` (Yet Another Resource Negotiator) is another core component of the `Apache Hadoop` ecosystem. It is a resource management and scheduling system used to coordinate and manage computing resources in a cluster.
### Opening the Configuration File
```bash
cd /path/to/hadoopfile/ # Enter the Hadoop installation directory
cd etc/hadoop/ # Enter the Hadoop configuration file directory
cd yarn-site.xml # Enter the YARN configuration file
```
* Note: /path/to/hadoopfile/ is the Hadoop installation directory, modify it as needed.
### Configuration File Content

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
* Note: hadoopname03 is the hostname of the ResourceManager, modify it as needed. hadoopname02 is the hostname of the log server, modify it as needed.
### Configuration Content Explanation
1. yarn.nodemanager.aux-services - mapreduce_shuffle This property defines the auxiliary services enabled on the YARN NodeManager. Here, mapreduce_shuffle specifies that the Shuffle process of MapReduce jobs will be handled by the NodeManager. Shuffle is a key step in MapReduce jobs, responsible for transferring data between the Map and Reduce phases. Enabling the mapreduce_shuffle service means the NodeManager will provide this functionality for MapReduce jobs.
2. yarn.resourcemanager.hostname - hadoopname03 This property specifies the hostname or address of the YARN ResourceManager. hadoopname03 is the hostname of the YARN resource manager, responsible for managing computing resources in the cluster, scheduling jobs, and coordinating task execution.
3. yarn.nodemanager.env-whitelist - JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME This property sets the whitelist of environment variables for the NodeManager. When the NodeManager starts, it reads and allows these environment variables from the whitelist. These variables usually point to Hadoop and Java related environment configurations, such as JAVA_HOME, HADOOP_COMMON_HOME, etc. This setting ensures that the NodeManager can access the correct environment variables to run and manage tasks properly.
4. yarn.log-aggregation-enable - true This property enables the YARN log aggregation feature. When set to true, YARN aggregates all log files of executed tasks to a centralized location instead of saving them on individual nodes. This helps simplify log management, especially when dealing with large-scale clusters.
5. yarn.log.server.url - `http://hadoopname01:19888/jobhistory/logs` This property specifies the URL address of the log server. `http://hadoopname01:19888/jobhistory/logs` is the Web interface address of the YARN log server, through which you can view the logs of jobs in the cluster. Users can access this address to view job execution logs, error messages, and detailed running status.
6. yarn.log-aggregation.retain-seconds - 604800 This property sets the retention duration for aggregated logs (in seconds). 604800 seconds is equivalent to 7 days. This means YARN will retain aggregated job logs for 7 days, and logs older than this will be automatically deleted. This setting helps administrators manage storage space and prevents log files from occupying disk space indefinitely.

## Configuring MapReduce
`MapReduce` is the core computing model of the `Apache Hadoop` ecosystem. It is a programming model and implementation framework for processing large-scale datasets. The design of MapReduce is inspired by functional programming, decomposing complex distributed computing problems into two main phases: Map and Reduce.
### Opening the Configuration File
```bash
cd /path/to/hadoopfile/ # Enter the Hadoop installation directory
cd etc/hadoop/ # Enter the Hadoop configuration file directory
cd mapred-site.xml # Enter the MapReduce configuration file
```
* Note: /path/to/hadoopfile/ is the Hadoop installation directory, modify it as needed.
### Configuration File Content

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
* Note: hadoopname01 is the hostname of the JobHistory server, modify it as needed. hadoopname01 is the hostname of the log server, modify it as needed.
### Configuration Content Explanation
1. mapreduce.framework.name - yarn This property specifies the execution framework for MapReduce jobs. Setting it to yarn means that MapReduce jobs will be scheduled and executed by the YARN (Yet Another Resource Negotiator) framework. YARN is the resource manager of Hadoop, used to dynamically allocate computing resources to different jobs.
2. mapreduce.jobhistory.address - hadoopname01:10020 This property specifies the address of the MapReduce JobHistory server. hadoopname01:10020 is the server address and port where job history information is saved. After a MapReduce job completes, the related job history information is recorded on this server for subsequent query and analysis.
3. mapreduce.jobhistory.webapp.address - hadoopname01:19888 This configuration defines the access address for the Web interface of the MapReduce JobHistory server. hadoopname01:19888 is the URL address of the JobHistory web application. You can access this address via a browser to view and manage historical data of completed MapReduce jobs, such as job status, execution time, task statistics, etc.

## Configuring core-site
`core-site.xml` is a core configuration file in the `Apache Hadoop` ecosystem, containing global configuration information for the Hadoop cluster. This file defines core Hadoop properties, such as the Hadoop name, data storage path, file system implementation class, etc.
### Opening the Configuration File
```bash
cd /path/to/hadoopfile/ # Enter the Hadoop installation directory
cd etc/hadoop/ # Enter the Hadoop configuration file directory
cd core-site.xml # Enter the core configuration file
```
* Note: /path/to/hadoopfile/ is the Hadoop installation directory, modify it as needed.
### Configuration File Content

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
* Note: hadoopname01 is the hostname of the NameNode, modify it as needed. /path/to/data is the data storage path, modify it as needed. byolio is the Hadoop username, modify it as needed.
### Configuration Content Explanation
1. fs.defaultFS - hdfs://hadoopname01:8020
This is the internal URI of the default file system for the Hadoop Distributed File System (HDFS). It specifies the NameNode server address and port (hadoopname01:8020). All file operations that do not explicitly specify a file system will use this address by default.
2. hadoop.tmp.dir - /path/to/data
This property sets the Hadoop temporary directory path. Hadoop stores various temporary files and intermediate data in this directory. This includes the local storage locations of the NameNode and DataNode, temporary files for MapReduce jobs, etc. It is recommended to set this directory on a disk with sufficient space.
3. hadoop.http.staticuser.user - byolio
This configuration defines the static username for the Hadoop Web UI (such as HDFS NameNode Web UI, ResourceManager Web UI, etc.). When accessing HDFS files through the Web interface, this user identity is used. This is important for proxy access and permission control.

## FAQ
### Why configure the JobHistory server functionality?
After a MapReduce job finishes running, the ResourceManager no longer saves the detailed information and logs of these jobs. It is necessary to configure the JobHistory server to save this information, allowing administrators and developers to view the execution details of completed jobs for troubleshooting errors.

### Why configure the log aggregation functionality?
Without enabling log aggregation, logs from each node are stored locally, making querying, accessing, and analyzing logs complex, especially when the cluster is large. After enabling log aggregation, logs from all jobs are stored centrally in one location, simplifying log searching and management.

### Why can't I perform delete operations on the Hadoop Web UI?
If a static username is not configured, the Hadoop Web UI will use the default dr.who user, which does not have permission to perform delete operations on the Hadoop Web UI. Therefore, it is necessary to configure the static username to a user with permission to perform such operations, so that administrators and developers can operate on the Hadoop Web UI.

### How to find default configuration files and selectively configure files?
Read the official documentation and examine the source code to find:
1. core-default.xml : Contains the default core configuration information for Hadoop, such as the Hadoop name, data storage path, file system implementation class, etc.
2. hdfs-default.xml : Contains the default configuration information for the Hadoop Distributed File System (HDFS), such as the NameNode server address and port, DataNode local storage location, MapReduce job temporary files, etc.
3. yarn-default.xml : Contains the default configuration information for the Hadoop resource management and scheduling system (YARN), such as the ResourceManager address and port, MapReduce job execution mode, log aggregation functionality, etc.
4. mapred-default.xml : Contains the default configuration information for the Hadoop MapReduce framework, such as the MapReduce job execution mode, log aggregation functionality, JobHistory server functionality, etc.
Based on these default configuration files, selectively configure the configuration information in core-site.xml, hdfs-site.xml, yarn-site.xml, and mapred-site.xml to achieve a customized Hadoop cluster configuration.

### What are the commonly used configuration ports?
Distributed systems involve a large number of services and processes. Without pre-defined ports, different services or processes might use the same port, leading to port conflicts, service unavailability, or system instability. Therefore, everyone uses some commonly used default ports for Hadoop 3.x:
NameNode internal communication port : 8020/9000/9820
Hadoop Web UI communication port : 9870
MapReduce Web UI port : 8088
JobHistory server communication port : 19888

## Summary
This article mainly introduced the basic configuration of Hadoop cluster configuration files, including the configuration of hdfs, yarn, mapreduce, core, and other configuration files. It provided the content of the configuration files and explanations of the configuration content, making it easier for you to understand the Hadoop cluster configuration files and better configure the Hadoop cluster to achieve high availability and high performance.
