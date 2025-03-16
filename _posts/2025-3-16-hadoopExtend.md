---
layout:     post
title:      "如何部署hadoop集群 (四)"
subtitle:   "让我们了解部分hadoop部署过程中的扩展知识"
date:       2025-3-16
author:     "Byolio"
header-img: "img/hadoopCopy.jpg"
catalog: true
tags:
    - linux
    - hadoop
---
> 本文主要介绍hadoop部署过程中的相关扩展内容, 以方便进一步进行接下来的hadoop部署操作

## 引言
我在[如何部署hadoop集群 (三)](https://byolio.top/2025/03/12/hadoopCopy/)中已经完成了对于虚拟机的克隆, 接下来我们需要了解一些额外的扩展知识, 以便于后续的hadoop集群部署操作

## 用户与用户权限
在hadoop集群启动过程中, 因为root用户的权限过大, 所以hadoop不允许我们直接使用root用户启动hadoop集群以保证hadoop集群安全, 因此我们需要使用普通用户来启动hadoop集群, 并且需要将普通用户赋予对hadoop的操作权限, 以便于普通用户可以使用hadoop集群
### 用户基础指令
在linux中, 我们可以使用以下指令来创建用户, 设置用户密码, 切换用户, 查看用户, 退出用户等操作
```bash
# 创建普通用户(在root下执行)
useradd username
# 设置普通用户密码(在root下执行)
passwd username
# 切换用户
su username
# 查看当前用户
whoami
# 退出当前用户
exit
```
* 注 : username为用户名称, 可以根据需要进行修改
### 如何解决hadoop集群因为使用root用户而启动失败的问题
先将自己移动到hadoop所在的目录下, 然后使用以下命令切换hadoop的所有者和组操作权限
```bash
chown -R username:usernameGroup hadoopdirectory
```
* 注 : username为用户名称, usernameGroup为用户组名称, hadoopdirectory为hadoop所在的目录名称, 可以根据需要进行修改
这样便可以赋予普通用户对hadoop文件夹的操作权限, 以便于普通用户可以使用hadoop集群
### 解决普通用户间的ssh免密登录问题
不同用户使用的ssh密钥是不同的, 因此我们需要将新建的普通用户的ssh公钥发送到hadoop集群中其他服务器用户, 以便于其他用户通过ssh连接该普通用户时可以进行免密登录, 具体指令可以查看之前的[内容](https://byolio.top/2025/03/12/hadoopCopy/#%E9%85%8D%E7%BD%AEssh%E5%85%8D%E5%AF%86%E7%99%BB%E5%BD%95), 将其中的`ssh-copy-id hostname`改为`ssh-copy-id username@hostname`即可
* 注 : username为用户名称, hostname为主机名, 可以根据需要进行修改, 只有部署集群的普通用户之间才需要进行此操作

## scp与rsync指令进行传输
scp和rsync是linux中常用的文件传输指令, 可以用于将文件从一台服务器传输到另一台服务器, 也可以用于将文件从一台服务器传输到本地, 也可以用于将文件从本地传输到另一台服务器, 具体指令如下 (使用前建议先配置ssh免密登录):
```bash
scp -r /home/user/source/ username@hostname:/path/to/destination
```
* 注 : source为源文件路径, username为目标服务器的用户名, hostname为目标服务器的主机名, /path/to/destination为目标文件路径
```bash
rsync -av /home/user/source/ username@hostname:/path/to/destination
```
* 注 : source为源文件路径, username为目标服务器的用户名, hostname为目标服务器的主机名, /path/to/destination为目标文件路径

## 如何关闭防火墙
hadoop集群在使用过程中的防火墙一般是加在hadoop集群整体上, 因此我们需要关闭每个服务器的防火墙, 以便于hadoop集群可以正常ping通使用, 具体指令如下:
```bash
systemctl stop firewalld
systemctl disable firewalld
```
* 注 : 以上指令需要在每个hadoop集群服务器上执行

## FAQ
### 切换用户与组后使用普通用户仍然存在权限问题
如果切换用户与组后使用普通用户仍然存在权限问题, 可以尝试使用以下指令来赋予该用户对hadoop文件夹的操作权限
```bash
chmod -R 777 hadoopdirectory
```
* 注 : hadoopdirectory为hadoop所在的目录名称, 可以根据需要进行修改, `777`为赋予所有人全部权限, 如不想赋予所有人全部权限, 可以根据需要进行修改
### 为什么有时候使用scp和rsync使用后会复制目录本身
scp和rsync其中的`/home/user/source/`中的最后一个斜杆不可以被删除, 否则会出现在复制目录中的文件的同时也复制了目录本身
### rsync是否支持在两个远程主机之间直接复制文件或目录
rsync不支持在两个远程主机之间直接复制文件或目录, 使用会报错, 因此如果需要在两个远程主机之间直接复制文件或目录, 需要使用scp指令
### rsync和scp有何区别
scp是覆盖复制, 即使目标文件已经存在, 也会覆盖目标文件, 而rsync是增量复制, 即如果目标文件已经存在, 则只会复制文件的变化部分, 不会覆盖目标文件 \
因此如果第一次使用scp复制文件, 后续如果修改了文件, 则使用rsync复制文件, 会将文件的变化部分复制到目标文件中, 而不会覆盖目标文件, 速度也会更加快