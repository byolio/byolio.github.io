---
layout: post
title: Two Basic Modes of RocketMQ
subtitle: 消息队列的两种基础模式的原理解释
date: 2024-12-3
author: Byolio
header-img: img/RocketMQMode.jpg
catalog: true
tags:
  - RocketMQ
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2024-12-3-RocketMQqueue/) | [English](https://www.byolio.blog/en/en/_posts/2024-12-3-RocketMQqueue/)

> This article aims to explain the principles of two basic modes of message queues.


## What is a Message Queue
A message queue is a technology used to pass messages in distributed systems. It provides an asynchronous, decoupled, and scalable way to handle messages. Message queues are typically used to decouple the dependency between producers and consumers, allowing them to develop and scale independently.

## Two Basic Modes of Message Queues
### 1. Queue Mode (Point-to-Point Mode)
It uses a queue as the data structure, where messages are processed in a First-In-First-Out (FIFO) order. Producers send messages to the queue, and consumers retrieve messages from the queue for processing. This mode is suitable for scenarios that require processing messages in order, such as task queues and workflows.
### 2. Publish-Subscribe Mode
It uses a one-to-many relationship between publishers and subscribers, where messages are published and subscribed to via a Topic. When a message is published to a topic, all consumers subscribed to that topic receive the message. This mode is suitable for scenarios that require decoupling the dependency between producers and consumers and allow multiple consumers to process messages simultaneously, such as event-driven architectures and message broadcasting.

## Advantages of Publish-Subscribe Mode over Queue Mode
Compared to queue mode, publish-subscribe mode has the following advantages:
1. In queue mode, once a message is consumed, it is deleted from the queue and cannot be consumed again. In publish-subscribe mode, consumers only move the message position when consuming, so the message is retained and can be consumed by multiple consumers simultaneously. This also avoids the need for multiple consumers to replicate multiple queues, which consumes significant storage resources.
2. Publish-subscribe mode uses a logging approach, allowing erroneous consumption to be skipped directly. If previously consumed content is lost, it can be re-consumed to fill in the gaps.
3. Publish-subscribe mode uses a Topic-Consumer Group-Queue structure, storing messages sequentially in queues. It speeds up reading by having each consumer in a consumer group read different queues or having one consumer read multiple queues simultaneously.

## Summary
The above is an explanation of the principles of the two basic modes of RocketMQ.
