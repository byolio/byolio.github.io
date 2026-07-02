---
layout: post
title: How to Deploy a Hadoop Cluster (Part 6)
subtitle: 让我们完成hadoop集群的初始化和启动与关闭和删除
date: 2025-3-30
author: Byolio
header-img: img/hadoopStart.jpg
catalog: true
tags:
  - linux
  - hadoop
lang: en
---

> 🌐 [中文](/2025-3-30-hadoopStart/) | [English](/en/en/_posts/2025-3-30-hadoopStart/)

> This article mainly introduces the final steps of the Hadoop deployment process: initialization, startup, shutdown, and deletion.

## Introduction
After the previous steps of virtual machine deployment and Hadoop cluster installation and configuration, we now need to complete the final steps of the Hadoop cluster: initialization, startup, shutdown, and deletion.

## Configure the workers Configuration File
First, add a configuration file to specify all nodes in the Hadoop cluster.
```bash
cd /path/to/hadoopfile/etc/hadoop/
vim workers
```
* Note: /path/to/hadoopfile/ is the Hadoop installation directory; modify it as needed.
Add the hostnames of all nodes, one per line, for example:
```bash
hadoopname01
hadoopname02
```

## Initialize HDFS
```bash
cd /path/to/hadoopfile/ # Enter the Hadoop installation directory
```
* Note: /path/to/hadoopfile/ is the Hadoop installation directory; modify it as needed.
Before starting HDFS for the first time, format the NameNode:
```bash
hdfs namenode -format 
```
* Note: Formatting the NameNode should be performed on the NameNode node; the Secondary NameNode (2NN) does not need to be formatted.

## Start the Cluster
### Start HDFS
First, switch to a non-root account. The reason was mentioned in a [previous article](https://byolio.top/2025/03/16/hadoopExtend/).
```bash
su - hadoopName # Switch to the Hadoop user
```
* Note: hadoopName is a non-root user; modify it as needed.
```bash
start-dfs.sh # Start HDFS
```
* Note: Starting HDFS should be performed on the NameNode node; other nodes such as the Secondary NameNode (2NN) do not need to execute this command.

### Start YARN
```bash
start-yarn.sh # Start YARN
```
* Note: Starting YARN should be performed on the ResourceManager node; other nodes do not need to execute this command.

### Start HistoryServer
```bash
/path/to/hadoopfile/bin/mapred --daemon start historyserver
```
* Note: /path/to/hadoopfile/ is the Hadoop installation directory; modify it as needed. Starting the HistoryServer should be performed on the JobHistoryServer node; other nodes do not need to execute this command.

### Check Cluster Status
```bash
jps
```
If you see processes other than jps running on the corresponding nodes, the cluster has started successfully.

## Shut Down the Cluster
### Shut Down HistoryServer
```bash
/path/to/hadoopfile/bin/mapred --daemon stop historyserver
```
* Note: /path/to/hadoopfile/ is the Hadoop installation directory; modify it as needed. Shutting down the HistoryServer should be performed on the JobHistoryServer node; other nodes do not need to execute this command.

### Shut Down YARN
```bash
stop-yarn.sh # Shut down YARN
```
* Note: Shutting down YARN should be performed on the ResourceManager node; other nodes do not need to execute this command.

### Shut Down HDFS
```bash
stop-dfs.sh 
```
* Note: Shutting down HDFS should be performed on the NameNode node; other nodes such as the Secondary NameNode (2NN) do not need to execute this command.

## Check Cluster Status
```bash
jps
```
If no processes other than jps are seen, the cluster has shut down successfully.

## Delete the Cluster
If configuration issues arise, you can delete the cluster and redeploy.
```bash
rm -rf /path/to/data
```
* Note: /path/to/data is the Hadoop data directory; modify it as needed. Delete the Hadoop data directory.
```bash
rm -rf /path/to/logs
```
* Note: /path/to/logs is the Hadoop log directory; modify it as needed. Delete the Hadoop log directory.

Then reinitialize and format HDFS.
* Note: Each DataNode node needs to delete its corresponding Hadoop data directory and log directory.

## FAQ
### Why does the cluster report errors after configuring workers?
1️⃣ Ensure that the hostnames of each node are correctly configured in the workers file, and also ensure that the corresponding hostnames and IP addresses are correctly configured in the /etc/hosts file.
2️⃣ Only one node hostname should be configured per line in the workers file; multiple hostnames cannot be configured on the same line.
3️⃣ There should be no spaces or comment lines.
### Why format the NameNode?
The main purpose of formatting the NameNode is to initialize or reinitialize HDFS metadata, including the file system directory structure, file block information, permissions, etc.
### Correct startup and shutdown order and why
#### Startup Order
1️⃣ Start NameNode (HDFS metadata management)
2️⃣ Start Secondary NameNode (merge metadata logs, optional)
3️⃣ Start DataNode (store HDFS data blocks)
4️⃣ Start ResourceManager (YARN task scheduling)
5️⃣ Start NodeManager (worker nodes for executing computing tasks)
6️⃣ Start JobHistory Server (store history of completed jobs)
#### Shutdown Order
1️⃣ Shut down JobHistory Server (ensure historical data is fully written)
2️⃣ Shut down NodeManager (prevent running tasks from being forcibly interrupted)
3️⃣ Shut down ResourceManager (prevent YARN from accepting new tasks)
4️⃣ Shut down DataNode (avoid HDFS misjudging data loss)
5️⃣ Shut down Secondary NameNode (avoid log merge errors after NameNode shutdown)
6️⃣ Shut down NameNode (shut down last to ensure metadata integrity)
#### Why follow this order?
1️⃣ HDFS must start first because YARN and JobHistory Server depend on HDFS.
2️⃣ DataNode must start after NameNode; otherwise, the NameNode will enter safe mode (if some DataNodes have not responded or HDFS data is inconsistent, the NameNode enters Safe Mode to prevent incorrect data from being written to the system).
3️⃣ JobHistory Server must start after YARN; otherwise, job history cannot be saved.
4️⃣ The shutdown order is the reverse to ensure metadata integrity and prevent the NameNode from misjudging data loss.
### Why delete the data and logs directories?
1️⃣ The old data directory stores previous block data, and formatting the NameNode reinitializes metadata. If old data still exists, it may cause mismatches between new metadata and actual storage, leading to errors or data inconsistency.
2️⃣ Log confusion: Old logs in the logs directory may confuse debugging information for the new system, causing trouble during troubleshooting.
