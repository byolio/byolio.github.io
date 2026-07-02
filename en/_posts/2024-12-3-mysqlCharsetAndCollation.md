---
layout: post
title: MySQL Character Sets and Collation Rules
subtitle: 系统性的聊一聊mysql字符集与比较规则
date: 2024-12-3
author: Byolio
header-img: img/MySQL.jpg
catalog: true
tags:
  - mysql
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2024-12-3-mysqlCharsetAndCollation/) | [English](https://www.byolio.blog/en/en/_posts/2024-12-3-mysqlCharsetAndCollation/)

> This article aims to systematically discuss MySQL character sets and collation rules.

## What are MySQL Character Sets and Collation Rules
Character set: A character set defines the characters stored and their encoding method. For example, `utf8mb4` is a character set that uses 4 bytes per character and is based on the Unicode standard, while `latin1` is a Western European character set that uses 1 byte per character and is based on the ISO-8859-1 standard.
Collation rule: A collation rule defines how strings are compared for sorting and equality based on a character set. For example, `utf8mb4_general_ci` (sorting method in MySQL 5.7, MySQL 8.0 primarily uses `utf8mb4_0900_ai_ci`) is a collation rule based on the `utf8mb4` character set, which is case-insensitive and accent-insensitive.


## Classification of Character Sets and Collation Rules
### Classification of Character Sets
1. Single-byte character sets: such as `latin1`, `cp1252`, etc., each character occupies 1 byte.
2. Multi-byte character sets: such as `utf8`, `utf16`, etc., each character occupies 1-4 bytes.
3. Variants of multi-byte character sets: such as `utf8`, `utf8mb4`, each character occupies 1-4 bytes, supporting more Unicode characters.
4. Non-Unicode character sets: such as `gbk`, `big5`, etc., each character occupies 2 bytes.

### Classification of Collation Rules
1. `_ai`: Accent Insensitive
2. `_as`: Accent Sensitive
3. `_ci`: Case Insensitive
4. `_cs`: Case Sensitive
5. `_bin`: Binary
6. `_general`: Loose collation rule (does not strictly follow Unicode standard)
7. `unicode`: Strict collation rule (strictly follows Unicode standard)
(Note: `_ai` and `_as` are collation rules for accents, `_ci` and `_cs` are for case sensitivity, `_bin` is for binary comparison. `_as_cs` has the advantage of locale-specific comparison compared to `_bin`.)

## Levels of Collation Rules
```sql
# 存在四个级别的字符集和比较规则: (从高到低)
# 1. 服务器级别  server
# 2. 数据库级别  database
# 3. 表级别      table
# 4. 列级别      column
```
If no character set and collation rule are specified, the lower-level character set and collation rule default to the higher-level ones.

## MySQL Practical Commands
1. View character sets and collation rules
```sql
SHOW variables like '%character%';;
```
2. View the character set and collation rule of the current database and table
```sql
SHOW CREATE DATABASE db_name;
SHOW CREATE TABLE table_name;
```
3. Modify character sets and collation rules
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
Because when designing a database, it is best practice to properly plan character sets, collation rules, and field default values from the beginning, which can reduce modification costs and potential risks in subsequent maintenance. Therefore, it is generally not recommended to modify the character sets and collation rules of databases and tables.
4. Specify character sets and collation rules at creation time
```sql
# 在创建数据库时指定字符集和比较规则
CREATE DATABASE db_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci;
# 在创建表时指定列的字符集和比较规则
CREATE TABLE table_name (
    column_name VARCHAR(255) CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci
);
```

## Summary
1. Character sets and collation rules are important concepts in MySQL database design; they determine how data is stored and compared.
2. When designing a database, choosing appropriate character sets and collation rules can improve database performance and reliability.
3. When modifying the character sets and collation rules of databases and tables, careful consideration must be given to data migration and compatibility issues.
4. When creating databases and tables, you can specify character sets and collation rules at creation time to ensure data consistency and reliability.
5. When modifying character sets and collation rules, pay attention to the difference between session level and global level. \
The above is my systematic discussion of MySQL character sets and collation rules.
