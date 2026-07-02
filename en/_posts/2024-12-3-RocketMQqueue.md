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

> 🌐 [中文](/2024-12-3-RocketMQqueue/) | [English](/en/en/_posts/2024-12-3-RocketMQqueue/)

> This article aims to explain the principles of two basic modes of message queues.


## What is a Message Queue
A message queue is a technology used to pass messages in distributed systems. It provides an asynchronous, decoupled, and scalable way to handle messages. Message queues are typically used to decouple the dependency between producers and consumers, allowing them to develop and scale independently.

## Two Basic Modes of Message Queues
### 1. Queue Mode (Point-to-Point Mode)
It uses a queue as the data structure, and messages are processed in a first-in-first-out (FIFO) order. Producers send messages to the queue, and consumers retrieve messages from the queue for processing. This mode is suitable for scenarios where messages need to be processed in order, such as task queues, workflows, etc.
### 2. Publish-Subscribe Mode
It uses a one-to-many relationship between publishers and subscribers, where messages are published and subscribed through a topic. When a message is published to a topic, all consumers subscribed to that topic receive the message. This mode is suitable for scenarios where the dependency between producers and consumers needs to be decoupled, and multiple consumers are allowed to process messages simultaneously, such as event-driven architectures, message broadcasting, etc.

## Advantages of Publish-Subscribe Mode over Queue Mode
The publish-subscribe mode has the following advantages over the queue mode:
1. In the queue mode, once a message is consumed, it is deleted from the queue and cannot be consumed again. In the publish-subscribe mode, consumers only move the message position when consuming, and the message is retained, allowing multiple consumers to consume it simultaneously. This also avoids the need for multiple consumers to replicate multiple queues, which consumes a large amount of storage resources.
2. The publish-subscribe mode uses a logging approach, allowing consumers to skip erroneous messages directly. If previously consumed content is lost, it can be replenished by re-consuming the content.
3. The publish-subscribe mode uses a Topic-Consumer Group-Queue structure, storing messages sequentially into queues. By having each consumer in a consumer group read different queues, or one consumer read multiple queues simultaneously, the reading speed is accelerated.

## Summary
The above is an explanation of the principles of the two basic modes of RocketMQ.
