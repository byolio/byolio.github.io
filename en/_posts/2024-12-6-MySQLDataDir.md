---
layout: post
title: Tables in MySQL Directory Structure
subtitle: Linux上MySQL目录结构中的表
date: 2024-12-6
author: Byolio
header-img: img/MySQLDir.jpg
catalog: true
tags:
  - mysql
  - linux
lang: en
---

> 🌐 [中文](/2024-12-6-MySQLDataDir/) | [English](/en/en/_posts/2024-12-6-MySQLDataDir/)

> This article aims to introduce the directory structure of tables in MySQL on Linux.


## Directory for Storing MySQL Tables
The directory structure for storing data in the database is as follows:
```bash
cd /var/lib/mysql
```
Select a database folder to enter and view the table files within it.

## Table Files for Different Engines
In the `InnoDB` engine,
1. In MySQL 5.7, the table file suffix is .ibd, and the table data file suffix is .frm.
2. In MySQL 8.0, the table file suffix is .ibd, and the table data file is also written into it.
You can use the `ibd2sdi` command to convert the .ibd file to a .txt file for viewing.
```bash
ibd2sdi --dump-file=table.txt table.ibd
```
In the `MyISAM` engine,
1. The files storing table data are .MYD (data) and .MYI (index).
2. The file storing the table structure is .frm.

## Independent Tablespace and System Tablespace
1. Independent tablespace (after MySQL 5.6.6):
   The tablespace file and table structure file are stored separately in the database folder.
2. System tablespace (before MySQL 5.6.6):
   The tablespace file and table structure file are stored together in a single file under `/var/lib/mysql`.
(You can add the `innodb_file_-per_table` parameter under the `[server]` section in the my.cnf file and assign a corresponding value: 0 means using system tablespace; 1 means using independent tablespace.)


## Views
A view is a virtual table that does not store data, so it only generates a .frm file.

## Summary
1. The suffix of table files depends on the storage engine type.
2. The difference between independent tablespace and system tablespace lies in the storage location of the tablespace file and the table structure file.
3. A view is a virtual table that does not store data, so it only generates a .frm file.
