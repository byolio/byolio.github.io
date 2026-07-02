---
layout: post
title: Some Linux Basics About RocketMQ
subtitle: 关于零拷贝和文件预热的linux基础知识
date: 2025-5-10
author: Byolio
header-img: img/zeroCopyAndFilePreheating.jpg
catalog: true
tags:
  - RocketMQ
  - linux
lang: en
---

> 🌐 [中文](/2025-5-10-zeroCopyAndFilePreheating/) | [English](/en/en/_posts/2025-5-10-zeroCopyAndFilePreheating/)

> This article mainly introduces the basics of Linux zero-copy and file preheating.

## Zero-Copy
When sending a file over the network, the following steps generally occur:
1. The process switches to kernel mode.
2. Read the file content from disk into the kernel buffer.
3. Copy the content from the kernel buffer to the user buffer.
4. Copy the content from the user buffer to the network buffer.
5. Send the content from the network buffer to the network.
6. The process switches back to user mode and releases resources.

Overall, this process is very time-consuming and resource-intensive because it requires multiple copies, i.e., a large number of buffered I/O operations. In some scenarios, the number of copies is indeed excessive, leading to the emergence of zero-copy.

### sendFile for Zero-Copy
The steps are:
1. The process switches to kernel mode.
2. Read the file content from disk into the kernel buffer.
3. Copy the content from the kernel buffer to the network buffer.
4. Send the content from the network buffer to the network.
5. The process switches back to user mode and releases resources.
As can be seen, in this process, there is no copying to the user buffer. Instead, the content is directly copied from the kernel buffer to the network buffer, and then sent to the network. This reduces two copy operations and improves efficiency.

### sendFile + gather for Zero-Copy
The steps are:
1. The process switches to kernel mode.
2. Read the file content from disk into the kernel buffer.
3. Send the content from the kernel buffer to the network.
4. The process switches back to user mode and releases resources.
As can be seen, in this process, data no longer needs to be copied to the socket buffer (network buffer). Instead, the content is directly sent from the kernel buffer to the network. This reduces three copy operations and improves efficiency.

### Scenarios Suitable for Zero-Copy
Scenarios where there is no need to parse, modify, or backup the disk file content are suitable for zero-copy. Since the data is not copied to user space, its content cannot be parsed.

## File Preheating
File preheating refers to loading a file into memory in advance to avoid reading from disk each time the file is accessed. This can significantly improve file read performance, especially in scenarios with frequently accessed files or high concurrency, effectively reducing I/O latency.

### mmap + File Preheating
When RocketMQ preheats files, mmap maps the kernel read buffer corresponding to the disk file and the user's cache to a single address. Operations on this cached data in user space can directly affect the kernel's read buffer. Essentially, it is a piece of physical memory, reducing copies from kernel space to user space, allowing user space to directly manipulate this data. \
However, physical memory does not actually allocate resources initially. Only when a process accesses it and finds no data in memory does a page fault occur, triggering resource allocation. This page fault is a system call involving context switching, which is time-consuming and can cause performance fluctuations for RocketMQ message writing operations. \
Therefore, RocketMQ adopts file preheating, which involves traversing each page of the currently mapped file in advance, writing a zero byte, triggering the system call for page faults, pre-allocating memory, and locking part or all of the process's address space in physical memory to prevent it from being swapped to swap space.

## Summary
Zero-copy and file preheating are important means to improve file read performance, reducing I/O latency and enhancing file read efficiency.
