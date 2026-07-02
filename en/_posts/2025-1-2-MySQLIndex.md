---
layout: post
title: MySQL Index
subtitle: 系统性的聊一聊mysql的索引
date: 2025-1-3
author: Byolio
header-img: img/MySQLIndex.jpg
catalog: true
tags:
  - mysql
lang: en
---

> 🌐 [中文](/2025-1-2-MySQLIndex/) | [English](/en/en/_posts/2025-1-2-MySQLIndex/)

> This article aims to systematically discuss MySQL indexes.

## What is an Index
An index is a `data structure` used to improve the efficiency of database queries. It can speed up SELECT query operations by creating indexes on table columns.

## Types of Indexes
1. **Clustered Index**: A clustered index stores data within the index itself; the index is the data, and the data is the index. It is used in the B+ tree index structure of the InnoDB storage engine in MySQL.
2. **Non-clustered Index**: Any index where data and index are stored separately is a non-clustered index, such as secondary indexes, non-clustered composite indexes, etc. For example, in the MyISAM storage engine, MYD and MYI files store data and indexes separately, using a B+ tree index structure but without storing data, instead using pointers to jump to the data.

## Indexes in Different Storage Engines
Here are the common storage engines:
* **InnoDB**: Always has one clustered index and can have multiple secondary indexes.
* **MyISAM**: Has multiple non-clustered indexes that use pointers to jump to data.
* **Memory**: Uses Hash indexes, with data stored in memory.
* **Archive**: Has only a single non-clustered index and does not support secondary indexes.
* **CSV**: Does not support any indexes.

## B+ Tree Index
The most commonly used storage engine in MySQL is InnoDB, which employs the B+ tree index structure.

The B+ tree is a multi-way balanced search tree that reflects the structural optimization from: Binary Search Tree -> AVL Tree -> B-Tree -> B+ Tree.
* The disadvantage of a Binary Search Tree is that nodes may be unevenly distributed, causing the tree to become skewed, leading to query efficiency approaching O(n) of a linked list. This led to the development of the AVL tree (balanced binary search tree).
* The disadvantage of the AVL tree is that it is not short and fat enough; it has too few branches, resulting in excessive height and still inefficient queries.
* The B-tree is a special M-ary tree where each node can store multiple data items and have multiple child nodes. However, its use of in-order traversal requires repeated node jumps to access data, and each node contains data that takes up space. This reduces the number of child nodes that can be pointed to when the data page is 16KB, increasing the tree's height.
* The B+ tree is a special B-tree where each node stores only indexes via Hash, while data is stored in leaf nodes (data pages). Leaf nodes are connected using a doubly linked list, and internal data is connected using a singly linked list. The height difference between leaf nodes does not exceed 1. Since the bottom nodes contain data that takes up space, the upper nodes can point to more child nodes when the data page is 16KB, reducing the tree's height.

With a data page size of 16KB, the B+ tree can theoretically handle 1 billion data items at 4 levels, with a time complexity of O(log N). The root node is stored in memory, while other nodes are stored on disk. Each query requires at most 3 disk I/O operations, and the height of the B+ tree is very stable, making it an excellent index structure.

## Hash Index
A Hash index is more efficient than a B+ tree index for equality queries, but its disadvantages are also significant:
* Range queries are inefficient and degrade to O(n).
* The hash value calculation method can cause hash collisions in columns with many duplicates, reducing query efficiency to near O(n) of a linked list.
* Hash combines the keys of a composite index by addition for calculation, so it cannot use the leftmost prefix principle like a B+ tree.
* Data stored via hash has no order, so ORDER BY requires re-sorting the data, making it unsuitable for sorting.

It is used in storage engines like Memory.

## Summary
The above is a systematic discussion of MySQL indexes.
