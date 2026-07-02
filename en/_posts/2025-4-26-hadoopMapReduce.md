---
layout: post
title: How to Execute MapReduce Tasks on Hadoop
subtitle: 如何通过IDEA完成项目导入与jar包的上传并在hadoop上完成MapReduce任务的执行
date: 2025-4-26
author: Byolio
header-img: img/MapReduce.jpg
catalog: true
tags:
  - linux
  - hadoop
  - java
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2025-4-26-hadoopMapReduce/) | [English](https://www.byolio.blog/en/en/_posts/2025-4-26-hadoopMapReduce/)

> This article mainly introduces how to import a project and upload a JAR file via IDEA, and then execute MapReduce tasks on Hadoop (requires a pre-written Java project and its corresponding JAR file).

## Introduction
After starting the Hadoop cluster as described earlier, we can begin writing MapReduce tasks. This article focuses on how to import a project module and required JAR files using IDEA, upload the JAR file to Linux, and execute MapReduce tasks on Hadoop.

## Import Project Module and Required JAR Files
### Import Project Module
Open IDEA, as shown in the image:
![IDEAProject](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/IDEAProject.png)
Click `File` -> `New` -> `Project from Existing Sources`, select the directory of the pre-written Java project, proceed step by step, choose JDK version 1.8 (Java 8), regenerate the `.iml` and `.idea` files, and finally click `Finish`.
### Import Required JAR Files
Click `File` -> `Project Structure`, as shown in the image:
![IDEAjar](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/IDEAjar.png)
Click `Libraries` -> `+` -> `Java`, select the required JAR files, and click `OK`.
### Import the Uploaded JAR Files into the Module
Click `File` -> `Project Structure` -> `Modules` -> `Dependencies`, click `+` -> `Library`, select the previously uploaded JAR files, and click `OK`.

## Package the Project into a JAR File
![completejar](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/completejar.png)
Click `File` -> `Project Structure` -> `Artifacts`, click `+` -> `JAR` -> `From modules with dependencies`, select the previously imported `Module` and `Main Class`, click `OK`, then click `Apply` -> `OK`.

A JAR file will then be generated in the `artifacts` directory under the `out` folder. You can upload this JAR file to Linux using Xftp.

## How to Execute the JAR File
First, upload the JAR file to Linux, then execute the following command on Linux:
```bash
hadoop jar /path/to/jarfile.jar /path/to/input/file /path/to/output 
```
* Note: `/path/to/jarfile.jar` is the path to the JAR file, `/path/to/input/file` is the input file, and `/path/to/output` is the output file path. Modify them as needed.

## View Output Results
Open the MapReduce web UI to see the historical execution records of MapReduce. You can determine whether the task was executed successfully, along with execution time and other information.

## FAQ

### Why is the /path/to/input/file not being read?
`/path/to/input/file` and `/path/to/output` are paths on HDFS. Therefore, you need to upload the file to HDFS first, then execute the command. The command is as follows:
```bash
hadoop fs -put /path/to/local/file /path/to/input/file
```
* Note: `/path/to/local/file` is the path to the local file, and `/path/to/input/file` is the path on HDFS. Modify them as needed. If the `/path/to/input/file` directory does not exist, you need to create it first with the following command:
```bash
hadoop fs -mkdir /path/to/input/file
```

### What if the JAR file was not generated or was accidentally deleted?
Click `Build` -> `Build Artifacts` -> `Rebuild` to regenerate the JAR file.

