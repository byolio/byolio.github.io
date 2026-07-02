---
layout: post
title: How to Set Up MySQL 8.0 on Linux
subtitle: 让我们快速在linux上搭建MySQL8.0
date: 2024-11-23T00:00:00.000Z
author: Byolio
header-img: img/linuxAndMysql.png
catalog: true
tags:
  - linux
  - mysql
lang: en
---

> 🌐 [中文](/2024-11-23-mysqlOnServer/) | [English](/en/en/_posts/2024-11-23-mysqlOnServer/)

> This article aims to set up MySQL 8.0 on a CentOS 7 server.

Before setting up, you need to have a virtual machine or server ready and possess some basic Linux knowledge.

## Why Use Linux to Deploy MySQL
I believe there are four main reasons:
1. `Closer to real-world scenarios`: Deploying MySQL on Linux is more similar to actual enterprise production environments compared to Windows, helping you get familiar with server deployment and operations in advance.
2. `High performance support`: Enterprise Linux servers usually have higher software and hardware configurations, allowing better testing of MySQL's performance. (This post is just a simple deployment, no enterprise-level server required.)
3. `Team collaboration`: Deploying MySQL on Linux allows team members to log in separately for server management and development.
4. `Centralized management`: Teams can set up a unified database environment on Linux, avoiding development issues caused by individual environment differences.

## How to Deploy MySQL 8.0 on Linux
Many people first think of using `Xftp`, `Cyberduck`, `FileZilla`, etc., to transfer RPM packages, but finding the corresponding RPM package is not a simple task. Therefore, in this post, I use a simpler and more efficient method: deploying the `MySQL` repository to download and install the `MySQL` RPM package (it's cool, isn't it).
1. Deploy MySQL repository command:
```bash
cd /opt
wget https://dev.mysql.com/get/mysql80-community-release-el7-7.noarch.rpm
sudo rpm -ivh mysql80-community-release-el7-7.noarch.rpm
```
(The RPM package should follow the format: software name + version number + release number + system version + hardware architecture + extension)
If `wget` is not available, execute the following code to download `wget`:
`yum install wget -y`

2. Deploy the core components of `MySQL` on the Linux system, including the `MySQL` service and necessary dependencies.
Before deployment, ensure these two dependencies exist:
```bash
rpm -qa | grep libaio
rpm -qa | grep net-tools
```
If the output is empty, install the dependencies:
```
yum install -y libaio
yum install -y net-tools
```
Pull the `MySQL` service and necessary dependencies:
```bash
wget http://example.com/mysql-community-common-8.0.25-1.el7.x86_64.rpm
wget http://example.com/mysql-community-client-plugins-8.0.25-1.el7.x86_64.rpm
wget http://example.com/mysql-community-libs-8.0.25-1.el7.x86_64.rpm
wget http://example.com/mysql-community-client-8.0.25-1.el7.x86_64.rpm
wget http://example.com/mysql-community-server-8.0.25-1.el7.x86_64.rpm
```
3. Install the `MySQL` service and necessary dependencies:
Since the components have dependencies, install them in the following order to avoid dependency errors:
```bash
rpm -ivh mysql-community-common-8.0.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-plugins-8.0.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-8.0.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.0.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-8.0.25-1.el7.x86_64.rpm
```
Since `MariaDB` is a fork of the `MySQL` source code, it may conflict with `MySQL`. If a conflict error with `mariadb-libs` occurs, execute the following code:
```bash
yum remove -y mariadb-libs
```
Then continue with the installation.

4. Start the `MySQL` service
```bash
systemctl start mysqld;
systemctl enable mysqld;
systemctl status mysqld;
```
5. Check if the installation is complete:
`mysql --version`

## Other Issues
### Database Initialization
1. View the initialization password by entering the following command:
`cat /var/log/mysqld.log`
Find `A temporary password is generated for root@localhost: *******`
(The `*******` is the initial password; copy it.)
2. Log in to MySQL:
`mysql -uroot -p`
Then paste the initial password to log in.
3. Change the password:
After logging in, you need to change the password to proceed. Execute the following code:
```sql
alter user 'root'@'localhost' identified by '******';
```
Here `******` is the new password.

### Remote Login
If you need to log in to `MySQL` remotely using a client, log in to `MySQL` and enter the following code to modify permissions:
```sql
USE mysql;
update user set host = '######' where user = 'root';
ALTER USER 'root'@'######' IDENTIFIED WITH mysql_native_password BY '******';
flush privileges;
```
(Where `######` is the address to grant permissions; for learning purposes, you can directly enter `%`. `******` is the new password.)
Then log in via the client.
### Permission Errors
If a permission modification error prevents you from logging into `MySQL`, you can execute the following commands to skip permission checks:
(To prevent the root user from damaging MySQL, you cannot directly use the root user to skip permissions and log in):
```bash
systemctl stop mysqld;
useradd -r -s /bin/false mysql
chown -R mysql:mysql /var/lib/mysql
su -s /bin/bash mysql
mysqld --skip-grant-tables &
```
Then log in directly to MySQL and modify permissions.

## Summary
The above is my method for setting up MySQL on Linux. If you have any questions, feel free to comment below.
