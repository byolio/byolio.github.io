---
layout: post
title: Implementation Principles of RocketMQ's Feature Messages
subtitle: 关于RocketMQ的三种特性消息实现原理的解释
date: 2024-12-21T00:00:00.000Z
author: Byolio
header-img: img/MessageImplementationPrinciple.jpg
catalog: true
tags:
  - RocketMQ
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2024-12-20-MessageImplementationPrinciple/) | [English](https://www.byolio.blog/en/en/_posts/2024-12-20-MessageImplementationPrinciple/)

> This article aims to explain the implementation principles of RocketMQ's messages, mainly including delayed messages, transactional messages, and ordered messages.

## What is a Message
A message is a medium for communication between systems, which can transmit data or commands in a distributed system. Message transmission is achieved through message middleware, such as RocketMQ, Kafka, etc. Message middleware helps systems achieve high-performance and high-reliability data transmission through asynchronous decoupling.

RocketMQ mainly supports the following message models:
1. Normal messages: the most common type, where the producer sends messages to a queue, and the consumer pulls messages from the queue for processing.
2. Delayed messages: allow messages to be consumed by consumers only after a specified time.
3. Transactional messages: support distributed transactions, achieving transactional consistency between message sending and business execution.
4. Ordered messages: ensure that messages are consumed by consumers in the order they were sent by the producer.
This article will mainly focus on the implementation principles of delayed messages, transactional messages, and ordered messages.

## Delayed Messages
Delayed messages are immediately delivered by the `producer`, rather than being sent to the Broker only after the time arrives. This puts pressure on the Broker, causing it to accumulate a large number of messages, smoothing peaks and filling valleys. To prevent queue message congestion in the Broker or offset jumping (due to the `principle of locality` for preloading), and to avoid needing too many 'alarms' to handle each message individually (which would degrade performance), RocketMQ directly defines 18 levels of delayed delivery: 1s, 5s, 10s, 30s, 1m, 2m, 3m, 4m, 5m, 6m, 7m, 8m, 9m, 10m, 20m, 30m, 1h, 2h. Then it establishes a shared scheduled task thread pool with 18 core threads to manage the delivery of all delayed messages. This way, delayed messages are also uniformly categorized and constrained, making management and scheduling easier.

To achieve this, RocketMQ creates a special Topic (`SCHEDULE_TOPIC_XXXX`) for storing delayed messages. Delayed messages are first sent to this Topic, which can reuse the commitLog. The messages are then distributed to the consumeQueue. Consumers do not subscribe to this Topic, so they cannot consume the message at this point. Then the thread pool periodically schedules messages in each queue under this Topic. Once a message expires, it is distributed to the original Topic for consumers to consume.

Each delay level corresponds to an independent task (DeliverDelayedMessageTimerTask) with a delay time of 100ms. These tasks are submitted to the thread pool. The content of these tasks is to obtain the corresponding consumeQueue based on the incoming QueueID, along with the corresponding offset. The Broker periodically saves the consumption offset of the consumeQueue under `SCHEDULE_TOPIC_XXXX`. If the thread pool (deliverExecutorService) size is sufficient (e.g., equal to or greater than the number of delay levels), these tasks may be processed in parallel.

If a task expires, a new task is immediately created and thrown into the thread pool with a delay time of 100ms. The task's parameters include the updated offset, so the thread continues to consume subsequent messages, and this cycle repeats. If the corresponding delayed message has not yet expired, the offset remains unchanged, and a new task is also immediately created and thrown into the thread pool. After 100ms, it will check again whether the message has expired.

From an implementation perspective, these methods greatly reduce the complexity of developing delayed messages, but such an implementation is `inaccurate in terms of delay time`. The execution time of tasks and the waiting time for queue consumption can both cause inaccuracies in the delay time.

## Transactional Messages
When it comes to transactions, it must be ACID (`Atomicity, Consistency, Isolation, Durability`). For data operations within the same database, the database's own mechanisms can be used to ensure transactions. However, for data operations across different databases, the database's own transactions cannot be used, and `distributed transactions` must be employed.

There are many solutions for implementing distributed transactions, such as 2PC, 3PC, TCC, local messages, and transactional messages. This article focuses on transactional messages.

Transactional messages are more suitable for asynchronous update scenarios, ensuring `eventual consistency`. The process allows for data inconsistency between the two ends.

At the beginning of a transaction, a half message is first sent to the Broker, which is an incomplete message. This type of message is skipped by consumers. Then, based on the result of the local transaction execution, a commit message or a rollback message is sent to the Broker. RocketMQ designs a check-back mechanism: the Broker checks with the producer through an interface exposed by the producer to determine whether the transaction was successful. In case of unexpected situations, the rollback message ensures transactional consistency. If the sending producer goes down, other producers in the same producer group can be checked by the Broker. This is one of the functions of the `producer group`.

When sending a half message, it is not sent to the original Topic, but to a specific Topic: `RMQ_SYS_TRANS_HALF_TOPIC`, with the default queue being 0. The Broker starts a scheduled thread service called `TransactionalMessageCheckService`, which periodically scans messages under the `RMQ_SYS_TRANS_HALF_TOPIC` Topic and requests the producer's check-back interface to see if the transaction was successful. If successful, it restores the original Topic for consumers to consume; if failed, it does not redeliver.

## Ordered Messages
Ordered messages need to guarantee orderliness in sending, storage, and consumption, meaning ensuring `temporal order` or `causal order` in these three stages.

In the sending phase, it is necessary to ensure that a `single producer` and `within a single producer, ordered messages are sent in a single thread (serially)`, to prevent order issues caused by multiple producers and multiple threads.

In the storage phase, it is necessary to ensure that related messages within a `single queue` are stored sequentially, so that related ordered messages are all assigned to the same consumeQueue. This is achieved by using the orderId modulo the number of queues, ensuring that the same order is always sent to the same queue, i.e., specifying the queue.

The consumption requirements are generally consistent with the sending phase. However, in business scenarios, if a previous message fails to be consumed, all related messages should also directly fail. These messages can then be persistently saved elsewhere for later repair and re-consumption. Additionally, consumer load balancing is involved.

In the consumption scenario, RocketMQ uses three common locks to ensure the orderliness of message consumption as much as possible.
1. Distributed lock: binds the corresponding queue to the consumer. If it is found that the queue has already been bound by another consumer, the message cannot be pulled for consumption. The consumer periodically (every 20s by default) renews this lock to ensure ownership of the distributed lock.
2. Synchronized: ensures that only one thread consumes this queue at the same time (allowing the same consumer to concurrently consume messages from different queues).
3. ReentrantLock: this lock acts more like a flag, indicating that there are still messages being consumed in this queue, preventing rebalancing until the next rebalance. (This lock has finer granularity.)

The dialectical relationship between orderliness and availability: if absolute orderliness is to be guaranteed, availability must be sacrificed, meaning messages cannot be sent to different queues. Conversely, if availability is to be guaranteed, absolute orderliness cannot be ensured.

RocketMQ provides solutions for both modes: if absolute orderliness is required, the `-o` parameter (--order) must be set to true when creating the Topic, and the configurations `orderMessageEnable` and `returnOrderTopicConfigToBroker` in the NameServer must be true. If any of the above conditions is false, availability is chosen.

## Summary
The above is my explanation of the implementation principles of RocketMQ's three feature messages.
