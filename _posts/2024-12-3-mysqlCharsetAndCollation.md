---
layout:     post
title:      "mysql字符集与比较规则"
subtitle:   "系统性的聊一聊mysql字符集与比较规则"
date:       2024-12-3
author:     "Byolio"
header-img: "img/MySQL.jpg"
catalog: true
tags:
    - mysql
---
> 本文旨在对mysql字符集与比较规则的系统性的聊一聊。

## 什么是mysql字符集与比较规则
字符集: 字符集定义了存储的字符及其编码方式。例如，`utf8mb4`是一种使用了4个字节为一个字符且基于Unicode字符标准的字符集，而`latin1`是一种使用了1个字节为一个字符基于ISO-8859-1标准的西欧字符集。
字符比较规则: 比较规则定义了在字符集的基础上，如何比较字符串的排序和大小。例如，`utf8mb4_general_ci`(mysql5.7排序方式, mysql8.0主要为utf8mb4_0900_ai_ci)是一种基于`utf8mb4`字符集的比较规则，它不区分大小写和重音符号。


## 字符集与比较规则的分类
### 字符集的分类
1. 单字节字符集: 如`latin1`、`cp1252`等，每个字符占用1个字节。
2. 多字节字符集: 如`utf8`、`utf16`等，每个字符占用1-4个字节。
3. 多字节字符集的变体: 如`utf8`, `utf8mb4`，每个字符占用1-4个字节，支持更多的Unicode字符。
4. 非Unicode字符集: 如`gbk`、`big5`等，每个字符占用2个字节。

### 比较规则的分类
1. `_ai`：Accent Insensitive（不区分重音）
2. `_as`：Accent Sensitive（区分重音）
3. `_ci`：Case Insensitive（不区分大小写）
4. `_cs`：Case Sensitive（区分大小写）
5. `_bin`：Binary（二进制比较）
6. `_general`: 比较规则较宽松 (不严格遵循Unicode标准)
7. `unicode`: 比较规则较严格 (严格遵循Unicode标准)
(注：`_ai`和`_as`是针对重音符号的比较规则，`_ci`和`_cs`是针对大小写的比较规则，`_bin`是针对二进制比较的比较规则, `_as_cs`相比于`_bin`具有语言当地语言比较的优势。)

## 比较规则的等级
```sql
# 存在四个级别的字符集和比较规则: (从高到低)
# 1. 服务器级别  server
# 2. 数据库级别  database
# 3. 表级别      table
# 4. 列级别      column
```
如果不进行字符集和比较规则的指定, 低级别的字符集和比较规则默认为高级别的字符集和比较规则

## MySQL实用指令
1. 查看字符集和比较规则
```sql
SHOW variables like '%character%';;
```
2. 查看当前数据库和表的字符集和比较规则
```sql
SHOW CREATE DATABASE db_name;
SHOW CREATE TABLE table_name;
```
3. 修改字符集和比较规则
```sql
# 修改数据库的字符集和比较规则
ALTER DATABASE db_name DEFAULT CHARACTER SET = utf8mb4 DEFAULT COLLATE = utf8mb4_general_ci;
# 修改表的字符集和比较规则
ALTER TABLE table_name CONVERT TO CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci;
# 修改列的字符集和比较规则
ALTER TABLE table_name MODIFY COLUMN column_name VARCHAR(255) CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci;
# 修改character_set_client, character_set_connection, character_set_results的字符集
(在会话层面修改字符集, global需要在配置文件中修改或直接在MySQL命令行中一个个指定修改)
SET NAMES utf8mb4;
# 指定修改会话字符集变量
SET @@session.character_set_client = utf8mb4;
# 指定修改会话比较规则变量
SET @@session.collation_connection = utf8mb4_general_ci;
# 指定修改全局字符集变量
SET GLOBAL character_set_server = utf8mb4;
# 指定修改全局比较规则变量
SET @@global.collation_server = utf8mb4_general_ci;
```
因为在设计数据库时，一开始就合理规划字符集、比较规则和字段默认值是最佳实践，这可以减少后续维护中的修改成本和潜在风险, 所以一般不建议修改数据库和表的字符集和比较规则。
4. 在创建时指定字符集和比较规则
```sql
# 在创建数据库时指定字符集和比较规则
CREATE DATABASE db_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci;
# 在创建表时指定列的字符集和比较规则
CREATE TABLE table_name (
    column_name VARCHAR(255) CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci
);
```

## 总结
1. 字符集和比较规则是MySQL数据库设计中的重要概念，它们决定了数据的存储和比较方式。
2. 在设计数据库时，选择合适的字符集和比较规则可以提高数据库的性能和可靠性。
3. 在修改数据库和表的字符集和比较规则时，需要谨慎考虑数据的迁移和兼容性问题。
4. 在创建数据库和表时，可以在创建时指定字符集和比较规则，以确保数据的一致性和可靠性。
5. 在修改字符集和比较规则时，需要注意会话级别和全局级别之间的区别。 \
以上就是我系统性的聊一聊MySQL字符集与比较规则的内容。
