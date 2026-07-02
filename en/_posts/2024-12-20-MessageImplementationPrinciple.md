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

> 🌐 [中文](/2024-12-20-MessageImplementationPrinciple/) | [English](/en/en/_posts/2024-12-20-MessageImplementationPrinciple/)

> This article aims to explain the implementation principles of RocketMQ messages, mainly including delayed messages, transactional messages, and ordered messages.

## What is a Message
A message is a medium used for communication between systems, capable of transmitting data or commands in a distributed system. Message delivery is achieved through message middleware, such as RocketMQ, Kafka, etc. Message middleware helps systems achieve high performance and high reliability data transmission through asynchronous decoupling.

RocketMQ mainly supports the following message models:
1. Normal Message: The most common type of message, where the producer sends the message to a queue, and the consumer pulls the message from the queue for processing.
2. Delayed Message: Allows the message to be consumed by the consumer only after a specified time.
3. Transactional Message: Supports distributed transactions, achieving transactional consistency between message sending and business execution.
4. Ordered Message: Ensures that messages are consumed by the consumer in the order they were sent by the producer.
This article will mainly focus on the implementation principles of delayed messages, transactional messages, and ordered messages.

## Delayed Message
A delayed message is immediately sent by the `producer`, not sent to the Broker by the `producer` after the time arrives. This puts pressure on the Broker, causing it to accumulate a large number of messages, smoothing out peaks and filling valleys. To prevent queue message congestion in the Broker or offset jumping around (`locality principle` for preloading) and the need for too many 'alarms' to process each message individually, which would degrade performance, RocketMQ directly defines 18 levels of delayed delivery: 1s, 5s, 10s, 30s, 1m, 2m, 3m, 4m, 5m, 6m, 7m, 8m, 9m, 10m, 20m, 30m, 1h, 2h. Then, a shared scheduled task thread pool is established, containing 18 core threads, to manage the delivery of all delayed messages. This way, delayed messages are also uniformly categorized and constrained, making management and scheduling easier.

To achieve this goal, RocketMQ creates a dedicated Topic for storing delayed messages (`SCHEDULE_TOPIC_XXXX`). Delayed messages are first sent to this Topic. This Topic can reuse the commitLog and distribute messages to the consumeQueue. Consumers do not subscribe to this Topic, so they cannot consume this message at this time. Then, the thread pool periodically schedules messages in each queue under this Topic. Once a message expires, it is distributed to the original Topic for consumption by consumers.

Each delay level corresponds to an independent task (DeliverDelayedMessageTimerTask), with a delay time of 100ms. These tasks are submitted to the thread pool. The content of these tasks is to obtain the corresponding consumeQueue based on the incoming QueueID, and of course, the corresponding offset. The Broker periodically saves the consumption offset of the consumeQueue under `SCHEDULE_TOPIC_XXXX`. If the thread pool (deliverExecutorService) size is sufficient (e.g., equal to or greater than the number of delay levels), these tasks may be processed in parallel.

If a task expires, a new task is immediately created and thrown into the thread pool with a delay time of 100ms. The task's parameters will include the updated offset, so the thread will continue to consume subsequent messages, and this cycle repeats. If the corresponding delayed message has not yet expired, the offset remains unchanged, and a new task is also immediately created and thrown into the thread pool. After 100ms, it will check again whether this message has expired.

From an implementation perspective, these methods greatly reduce the complexity of developing delayed messages, but such an implementation is `inaccurate for the delay time`. The execution time of the task and the waiting time for queue consumption can both cause inaccuracies in the delay time.

## Transactional Message
When it comes to transactions, it is definitely ACID (`Atomicity, Consistency, Isolation, Durability`). For data operations within the same database, the database's own mechanisms can be used to ensure transactions. However, for data operations belonging to different databases, the database's own transactions cannot be used, and `distributed transactions` must be employed.

There are many solutions for implementing distributed transactions, such as: 2PC, 3PC, TCC, Local Message, Transactional Message. This article discusses transactional messages.

Transactional messages are more suitable for asynchronous update scenarios, used to ensure `eventual consistency`. The process allows for data inconsistency between the two ends.

At the beginning of a transaction, a half message is first sent to the Broker. This is an incomplete message that will be skipped by consumers. Then, based on the execution result of the local transaction, it is decided whether to send a commit message or a rollback message to the Broker. RocketMQ designs a check-back mechanism: The Broker will check back with the producer through the interface exposed by the producer to see if the transaction was successful. In case of unexpected situations, a rollback message is used to ensure transaction consistency. If the sending producer goes down, other producers in the same producer group can be checked back by the Broker. This is one of the functions of the `producer group`.

When sending a half message, it is not sent to the original Topic but to a specific Topic: `RMQ_SYS_TRANS_HALF_TOPIC`, with the default queue being 0. The Broker starts a scheduled thread service `TransactionalMessageCheckService`, which periodically scans the messages under the `RMQ_SYS_TRANS_HALF_TOPIC` Topic, requests the producer's check-back interface to see if the transaction was successful. If successful, it restores the original Topic for consumer consumption; if failed, it does not redeliver.

## Ordered Message
Ordered messages need to ensure orderliness in sending, storage, and consumption. That is, ensuring `temporal order` or `causal order` in these three stages.

In the sending stage, it is necessary to ensure `single producer` and `single-threaded (serial) sending of ordered messages within a single producer` to prevent order issues caused by multiple producers and multiple threads.

In the storage stage, it is necessary to ensure that related messages within a `single queue` are stored in order, so that related ordered messages are all assigned to the same consumeQueue. The method is to use orderId modulo the number of queues so that the same order is always sent to the same queue, specifying the queue.

The consumption requirements are generally consistent with the sending stage. However, in business processing of ordered messages, if the previous message consumption fails, all related messages should directly fail. These messages can then be persistently saved elsewhere for later repair and re-consumption. Also, there is the consumer's load balancing.

For consumption scenarios, RocketMQ uses three common locks to ensure the orderliness of message consumption as much as possible.
1. Distributed Lock: Binds the corresponding queue to the consumer. If it is found that the queue has already been bound by another consumer, then messages cannot be pulled for consumption. The consumer will periodically (default every 20s) renew this lock to ensure ownership of the distributed lock.
2. Synchronized: Ensures that only one thread consumes this queue at a time (allowing the same consumer to concurrently consume messages from different queues).
3. ReentrantLock: This lock acts more like a flag, indicating that there are still messages being consumed in this queue, preventing rebalancing, and waiting for the next rebalance. (This lock has finer granularity)

The dialectical relationship between orderliness and availability: To ensure absolute orderliness, availability must be sacrificed, meaning messages cannot be sent to different queues. Conversely, to ensure availability, absolute orderliness cannot be guaranteed.

RocketMQ provides solutions for both modes: To achieve absolute orderliness, when creating a Topic, the -o parameter (--order) must be set to true, and the configurations orderMessageEnable and returnOrderTopicConfigToBroker in the NameServer must be true. If any of the above conditions is false, availability is chosen.


## Summary
The above is my explanation of the implementation principles of RocketMQ's three characteristic messages.
