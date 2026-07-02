---
layout: post
title: How to Deploy a Hadoop Cluster (Part 4)
subtitle: 让我们了解部分hadoop部署过程中的扩展知识
date: 2025-3-16
author: Byolio
header-img: img/hadoopExtend.jpg
catalog: true
tags:
  - linux
  - hadoop
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2025-3-16-hadoopExtend/) | [English](https://www.byolio.blog/en/en/_posts/2025-3-16-hadoopExtend/)

> This article mainly introduces related extended content during the Hadoop deployment process to facilitate further Hadoop deployment operations.

## Introduction
In [How to Deploy a Hadoop Cluster (Part 3)](https://byolio.top/2025/03/12/hadoopCopy/), I have completed the cloning of virtual machines. Next, we need to learn some additional extended knowledge to facilitate subsequent Hadoop cluster deployment operations.

## Users and User Permissions
During the startup of a Hadoop cluster, because the root user has excessive permissions, Hadoop does not allow us to directly use the root user to start the Hadoop cluster to ensure its security. Therefore, we need to use a regular user to start the Hadoop cluster and grant the regular user operational permissions on Hadoop so that the regular user can use the Hadoop cluster.

### Basic User Commands
In Linux, we can use the following commands to create a user, set a user password, switch users, view the current user, exit the current user, etc.
```bash
# Create a regular user (execute as root)
useradd username
# Set the password for the regular user (execute as root)
passwd username
# Switch user
su username
# View current user
whoami
# Exit current user
exit
```
* Note: `username` is the user name, which can be modified as needed.

### How to Solve the Problem of Hadoop Cluster Failing to Start Due to Using the Root User
First, navigate to the directory where Hadoop is located, then use the following command to change the owner and group permissions of Hadoop:
```bash
chown -R username:usernameGroup hadoopdirectory
```
* Note: `username` is the user name, `usernameGroup` is the user group name, and `hadoopdirectory` is the directory name where Hadoop is located. They can be modified as needed.

This grants the regular user operational permissions on the Hadoop folder, allowing the regular user to use the Hadoop cluster.

### Solving SSH Passwordless Login Between Regular Users
Different users use different SSH keys. Therefore, we need to send the SSH public key of the newly created regular user to users on other servers in the Hadoop cluster, so that other users can log in without a password when connecting to this regular user via SSH. For specific commands, refer to the previous [content](https://byolio.top/2025/03/12/hadoopCopy/#%E9%85%8D%E7%BD%AEssh%E5%85%8D%E5%AF%86%E7%99%BB%E5%BD%95), and change `ssh-copy-id hostname` to `ssh-copy-id username@hostname`.
* Note: `username` is the user name, `hostname` is the host name. They can be modified as needed. This operation is only required between regular users deploying the cluster.

## Transferring with scp and rsync Commands
scp and rsync are commonly used file transfer commands in Linux. They can be used to transfer files from one server to another, from a server to the local machine, or from the local machine to another server. The specific commands are as follows (it is recommended to configure SSH passwordless login before use):
```bash
scp -r /home/user/source/ username@hostname:/path/to/destination
```
* Note: `source` is the source file path, `username` is the user name of the target server, `hostname` is the host name of the target server, `/path/to/destination` is the destination file path.
```bash
rsync -av /home/user/source/ username@hostname:/path/to/destination
```
* Note: `source` is the source file path, `username` is the user name of the target server, `hostname` is the host name of the target server, `/path/to/destination` is the destination file path.

## How to Disable the Firewall
During the use of a Hadoop cluster, the firewall is generally applied to the entire cluster. Therefore, we need to disable the firewall on each server so that the Hadoop cluster can ping and communicate normally. The specific commands are as follows:
```bash
systemctl stop firewalld
systemctl disable firewalld
```
* Note: The above commands need to be executed on each Hadoop cluster server.

## FAQ
### Permission Issues Still Exist When Using a Regular User After Changing User and Group
If permission issues still exist when using a regular user after changing the user and group, you can try using the following command to grant the user operational permissions on the Hadoop folder:
```bash
chmod -R 777 hadoopdirectory
```
* Note: `hadoopdirectory` is the directory name where Hadoop is located. It can be modified as needed. `777` grants full permissions to everyone. If you do not want to grant full permissions to everyone, you can modify it as needed.

### Why Do scp and rsync Sometimes Copy the Directory Itself?
The trailing slash in `/home/user/source/` in scp and rsync commands must not be removed; otherwise, it will copy the directory itself along with the files inside it.

### Does rsync Support Directly Copying Files or Directories Between Two Remote Hosts?
rsync does not support directly copying files or directories between two remote hosts; using it will result in an error. Therefore, if you need to directly copy files or directories between two remote hosts, you need to use the scp command.

### What is the Difference Between rsync and scp?
scp performs an overwrite copy; even if the target file already exists, it will overwrite it. rsync performs an incremental copy; if the target file already exists, it only copies the changed parts of the file without overwriting the target file. \
Therefore, if you use scp to copy files for the first time, and later modify the files, using rsync to copy the files will copy only the changed parts to the target file without overwriting it, and the speed will be faster.
