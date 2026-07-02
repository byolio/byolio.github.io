---
layout:     post
title:      "如何在linux上搭建MySQL8.0"
subtitle:   "让我们快速在linux上搭建MySQL8.0"
date:       2024-11-23
author:     "Byolio"
header-img: "img/linuxAndMysql.png"
catalog: true
tags:
    - linux
    - mysql
---
> 本文旨在centos7服务器上搭建MySQL8.0

搭建之前需要部署好一台虚拟机或服务器, 并拥有一定linux相关知识。

## 为什么使用linux部署MySQL
我认为有以下四点原因: 
1. `更接近真实场景`：linux上部署MySQL环境相比于windows更贴近企业真实生产环境，有助于提前熟悉服务器部署及运维工作。
2. `高性能支持`：企业使用的linux服务器通常具有更高的软件和硬件配置，可以更好的测试MySQL的性能表现。(本post只是做一个简单的部署, 不需要企业级服务器)
3. `团队合作`: linux上部署MySQL可以用于团队分开登陆服务器管理和开发。
4. `集中管理`：团队可以通过统一的linux搭建数据库环境，避免个人环境差异导致的开发问题。

##  如何在linux上部署MySQL8.0
很多人第一时间会想到使用`Xftp`, `Cyberduck`, `FileZilla`等传输rpm包, 但找寻对应的rpm包本身就不是一个简单的事, 所以我在本贴使用更为简单高效的: 部署`MySQL源`的方式下载`MySQL`rpm包并安装(it's cool, isn't it)。
1. 部署MySQL源指令:
```bash
cd /opt
wget https://dev.mysql.com/get/mysql80-community-release-el7-7.noarch.rpm
sudo rpm -ivh mysql80-community-release-el7-7.noarch.rpm
```
(rpm包应该满足格式: 软件名称 + 版本号 + 发布次数 + 系统版本 + 硬件架构平台 + 扩展名)
如果不存在wget的话, 执行以下代码下载`wget`:
`yum install wget -y`


2. 在linux系统上部署`MySQL`的核心组件，包括`MySQL`服务和必要的依赖项。
部署之前需要确保这两项依赖项存在
```bash
rpm -qa | grep libaio
rpm -qa | grep net-tools
```
如果返回为空, 安装依赖项:
```
yum install -y libaio
yum install -y net-tools
```
拉取`MySQL`服务和必要的依赖项。:
```bash
wget http://example.com/mysql-community-common-8.0.25-1.el7.x86_64.rpm
wget http://example.com/mysql-community-client-plugins-8.0.25-1.el7.x86_64.rpm
wget http://example.com/mysql-community-libs-8.0.25-1.el7.x86_64.rpm
wget http://example.com/mysql-community-client-8.0.25-1.el7.x86_64.rpm
wget http://example.com/mysql-community-server-8.0.25-1.el7.x86_64.rpm
```
3. 安装`MySQL`的服务和必要的依赖项:
由于各个组件之间存在依赖, 因此按照以下顺序进行安装, 以防发生依赖关系错误:
```bash
rpm -ivh mysql-community-common-8.0.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-plugins-8.0.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-8.0.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.0.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-8.0.25-1.el7.x86_64.rpm
```
由于`MariaDB`是基于`MySQL`源代码分支`fork`而来的，会与`MySQL`发生冲突, 因此如果出现与`mariadb-libs`冲突错误, 请执行以下代码:
```bash
yum remove -y mariadb-libs
```
然后在继续完成安装。

4. 启动`MySQL`服务
```bash
systemctl start mysqld;
systemctl enable mysqld;
systemctl status mysqld;
```
5. 查看安装是否完成:
`mysql --version`

## 其他问题
### 数据库初始化
1. 查看初始化密码, 输入以下指令: \
`cat /var/log/mysqld.log` \
找到`A temporary password is generated for rootalocalhost: *******`
(这里的`*******`就是初始密码, 将它复制下来) 
2. 登陆mysql: \
`mysql -uroot -p` \
然后粘贴初始化密码登陆。
3. 修改密码: \
进入后需要修改密码才允许继续操作, 执行以下代码:
```sql
alter user root'@'localhost' identified by '******';
```
这里的`******`为修改后的密码。

### 远程登陆
如果在远程使用客户端登陆`MySQL`, 需要登陆`MySQL`后输入以下代码修改权限:
```sql
USE mysql;
update user set host = '######' where user = 'root';
ALTER USER 'root'@'######' IDENTIFIED WITH mysql_native_password BY '******';
flush privileges;
```
(其中`######`为给予权限的地址, 如果仅用于学习可以直接输入: `%`, `******`为修改后的密码)
让后登陆客户端即可。
### 权限错误
如果`MySQL`权限修改有误导致无法登陆`MySQL`, 可以执行以下指令跳过权限检查
(为防止root用户破坏mysql因此不能直接使用root用户跳过权限登陆):
```bash
systemctl stop mysqld;
useradd -r -s /bin/false mysql
chown -R mysql:mysql /var/lib/mysql
su -s /bin/bash mysql
mysqld --skip-grant-tables &
```
然后登陆直接登陆mysql修改权限即可。

## 总结
以上就是我关于Linux上建立MySQL的方法, 如有问题欢迎在下面评论区评论。
