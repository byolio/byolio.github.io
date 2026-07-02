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

> 🌐 [中文](/2024-12-3-mysqlCharsetAndCollation/) | [English](/en/en/_posts/2024-12-3-mysqlCharsetAndCollation/)

> This article aims to systematically discuss MySQL character sets and collation rules.

## What are MySQL Character Sets and Collation Rules
Character set: A character set defines the characters to be stored and their encoding method. For example, `utf8mb4` is a character set that uses 4 bytes per character and is based on the Unicode character standard, while `latin1` is a Western European character set that uses 1 byte per character and is based on the ISO-8859-1 standard.
Collation rules: Collation rules define how strings are compared for sorting and case sensitivity based on a character set. For example, `utf8mb4_general_ci` (sorting method in MySQL 5.7, MySQL 8.0 primarily uses `utf8mb4_0900_ai_ci`) is a collation rule based on the `utf8mb4` character set, which is case-insensitive and accent-insensitive.

## Classification of Character Sets and Collation Rules
### Classification of Character Sets
1. Single-byte character sets: e.g., `latin1`, `cp1252`, each character occupies 1 byte.
2. Multi-byte character sets: e.g., `utf8`, `utf16`, each character occupies 1-4 bytes.
3. Variants of multi-byte character sets: e.g., `utf8`, `utf8mb4`, each character occupies 1-4 bytes, supporting more Unicode characters.
4. Non-Unicode character sets: e.g., `gbk`, `big5`, each character occupies 2 bytes.

### Classification of Collation Rules
1. `_ai`: Accent Insensitive
2. `_as`: Accent Sensitive
3. `_ci`: Case Insensitive
4. `_cs`: Case Sensitive
5. `_bin`: Binary comparison
6. `_general`: Loose collation rules (not strictly following Unicode standards)
7. `unicode`: Strict collation rules (strictly following Unicode standards)
(Note: `_ai` and `_as` are collation rules for accents, `_ci` and `_cs` are for case sensitivity, `_bin` is for binary comparison, and `_as_cs` has advantages over `_bin` in terms of locale-specific comparison.)

## Levels of Collation Rules
```sql
# There are four levels of character sets and collation rules: (from high to low)
# 1. Server level
# 2. Database level
# 3. Table level
# 4. Column level
```
If no character set or collation rule is specified, the lower level defaults to the higher level's character set and collation rule.

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
# Modify the character set and collation rule of a database
ALTER DATABASE db_name DEFAULT CHARACTER SET = utf8mb4 DEFAULT COLLATE = utf8mb4_general_ci;
# Modify the character set and collation rule of a table
ALTER TABLE table_name CONVERT TO CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci;
# Modify the character set and collation rule of a column
ALTER TABLE table_name MODIFY COLUMN column_name VARCHAR(255) CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci;
# Modify the character sets of character_set_client, character_set_connection, character_set_results
(Modify character set at session level; global changes need to be made in the configuration file or specified one by one in the MySQL command line)
SET NAMES utf8mb4;
# Specify modification of session character set variable
SET @@session.character_set_client = utf8mb4;
# Specify modification of session collation variable
SET @@session.collation_connection = utf8mb4_general_ci;
# Specify modification of global character set variable
SET GLOBAL character_set_server = utf8mb4;
# Specify modification of global collation variable
SET @@global.collation_server = utf8mb4_general_ci;
```
Because it is best practice to plan character sets, collation rules, and field default values properly from the beginning when designing a database, which reduces modification costs and potential risks in subsequent maintenance, it is generally not recommended to modify the character set and collation rule of a database or table.
4. Specify character sets and collation rules during creation
```sql
# Specify character set and collation rule when creating a database
CREATE DATABASE db_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci;
# Specify character set and collation rule for a column when creating a table
CREATE TABLE table_name (
    column_name VARCHAR(255) CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci
);
```

## Summary
1. Character sets and collation rules are important concepts in MySQL database design, determining how data is stored and compared.
2. Choosing appropriate character sets and collation rules during database design can improve database performance and reliability.
3. When modifying the character set and collation rule of a database or table, careful consideration must be given to data migration and compatibility issues.
4. When creating a database or table, character sets and collation rules can be specified at creation time to ensure data consistency and reliability.
5. When modifying character sets and collation rules, attention should be paid to the difference between session level and global level.

The above is my systematic discussion of MySQL character sets and collation rules.
