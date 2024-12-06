---
layout:     post
title:      "MySQL目录结构中的表"
subtitle:   "Linux上MySQL目录结构中的表"
date:       2024-12-6
author:     "Byolio"
header-img: "img/MySQLDir.jpg"
catalog: true
tags:
    - mysql
    - linux
---
> 本文旨在介绍Linux上MySQL关于表的目录结构。


## MySQL存放表的目录
数据库中存放数据的结构目录如下：
```bash
cd /var/lib/mysql
```
选择一个数据库文件夹进入，查看其中的表文件

## 不同引擎的表文件
在`InnoDB`引擎中,
1. MySQL5.7中表文件的后缀名为.ibd，表数据文件的后缀名为.frm
2. MySQL8.0中表文件的后缀名为.ibd，表数据文件也被写入其中
可以使用`ibd2sdi`命令将.ibd文件转换为.txt文件查看
```bash
ibd2sdi --dump-file=table.txt table.ibd
```
在`MyISAM`引擎中，
1. 存放表文件的文件为.MYD(数据)和.MYI(索引)
2. 存放表结构的文件为.frm

## 独立表空间和系统表空间
1. 独立表空间(MySQL5.66之后):
   表空间文件和表结构文件分开存放在数据库的文件夹下
2. 系统表空间(MySQL5.66之前):
    将表空间文件和表结构文件放在一起存放在`/var/lib/mysql`下一个文件中
(可以修改my.cnf文件中的`innodb_file_-per_table`参数, 0:代表使用系统表空间；1：代表使用独立表空间)


## 视图
视图是一个虚拟的表，它不存储数据，因此其仅仅会生成一个.frm文件

## 总结
1. 表文件的后缀名取决于存储引擎类型
2. 独立表空间和系统表空间的区别在于表空间文件和表结构文件的存放位置不同
3. 视图是一个虚拟的表，它不存储数据，因此其仅仅会生成一个.frm文件
