---
layout: post
title: Introduction and Usage of the Disruptor Framework
subtitle: Disruptor框架解决高并发处理团队消息问题
date: 2026-6-18
author: Byolio
header-img: img/disruptor.png
catalog: true
tags:
  - java
  - Disruptor
  - JUC
  - SpringBoot
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2026-6-18-Disruptor/) | [English](https://www.byolio.blog/en/en/_posts/2026-6-18-Disruptor/)

## Introduction
When building multi-player team collaboration or real-time chat systems using WebSocket, a performance bottleneck often arises: when a WebSocket thread receives a team message, if it synchronously processes time-consuming business logic (such as database persistence, permission verification, broadcast distribution, etc.) directly in the current thread, the thread will be occupied for a long time.

In high-concurrency scenarios, this synchronous processing method quickly exhausts the network thread pool, causing subsequent team messages to be blocked, and even leading to connection timeouts and message loss, causing the system's overall throughput to plummet.

To solve this problem, we need to introduce an **asynchronous decoupling** mechanism. Among the many queue solutions, the **LMAX Disruptor** framework, with its extreme performance, has become our preferred tool for achieving efficient, low-latency team message processing.

## Introduction to Disruptor
Disruptor is a high-performance, low-latency **in-memory concurrency framework** developed by the British foreign exchange trading company LMAX. Its original design goal was to address the low-latency and high-throughput requirements of financial trading systems in a multi-threaded environment.

In traditional Java concurrent programming, we usually use `ArrayBlockingQueue` or `LinkedBlockingQueue` to implement the producer-consumer model. However, Disruptor takes a different approach, abandoning the traditional queue structure and achieving the terrifying performance of processing millions of orders per second with almost no explicit locks (Lock-Free).

### 1. Performance Pain Points of Traditional Queues
Before understanding why Disruptor is fast, we need to know the three major pain points of traditional concurrent queues (such as `BlockingQueue`) under high concurrency:
1. **Severe Lock Contention**: Traditional blocking queues usually rely on heavy locks like `ReentrantLock` or `synchronized` to ensure thread safety during enqueue (Put) and dequeue (Take) operations. Under high concurrency, the overhead of thread context switching is enormous.
2. **False Sharing Problem**: In traditional linked list or array structures, the head and tail pointers of the queue are often very close in memory. When multi-core CPUs read them, they load them into the same cache line, causing frequent cache invalidation between cores, triggering writes back to main memory, and severely slowing down CPU efficiency.
3. **Garbage Collection (GC) Pressure**: Frequent enqueue and dequeue operations are accompanied by the creation and destruction of a large number of objects, which can easily cause the JVM to trigger GC frequently, leading to system pauses (Stop-The-World).

### 2. Core Black Technology of Disruptor
To squeeze the last drop of performance out of the CPU, Disruptor introduces the following core designs:

#### ① Ring Buffer Structure (RingBuffer)
The core of Disruptor uses a fixed-size ring array called `RingBuffer`.
* **No GC Pressure**: The array is filled with empty data objects (Events) during initialization. During actual runtime, producers only modify the methods of these existing objects instead of creating new ones. This achieves **memory reuse** and generates almost no new GC.
* **Bitwise Addressing**: By setting the array size to a power of 2 (e.g., 1024), finding the next available position can use bitwise operations (`sequence & (length - 1)`) instead of modulo operations, which is extremely efficient.

#### ② Innovative Lock-Free Design: Sequence
Disruptor does not use traditional head and tail pointers. Instead, it allows each producer and consumer to maintain its own `Sequence` (a sequence number, essentially an incrementing `long` value).
* Threads determine whether there is new data to read or space to write by comparing their respective `Sequence` values.
* Internally, it uses low-level **Memory Barriers** to prevent CPU instruction reordering and **CAS (Compare And Swap)** atomic operations to replace traditional explicit locks, achieving complete lock-free operation.

#### ③ Solving False Sharing: Cache Line Padding

In the underlying implementation of core components (such as `Sequence`), Disruptor artificially pads 7 useless `long` variables before and after the core variable. This ensures that the core sequence number can **exclusively occupy a 64-byte CPU cache line**, completely avoiding the false sharing problem and maximizing the cache hit rate of multi-core CPUs.


## How to Import the Disruptor Framework in SpringBoot
Import the disruptor dependency in pom.xml:
```xml
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.4.2</version>
</dependency>
```
## Basic Usage Example of Disruptor
The following is a simple example showing how to use Disruptor to implement a producer-consumer model:
1. Define the Event

The Event is the data carrier that flows in the RingBuffer. Disruptor pre-instantiates these Events at startup.
```java
@Data
public class TeamMessageEvent {
    private String messagePayload;     // Message content (JSON string or text)
    private WebSocketSession session;  // WebSocket session of the sender
    private String teamId;             // Team/Room ID
    private String userId;             // User ID
}
```
1. Define the Consumer (WorkHandler)

The consumer is responsible for fetching data from the RingBuffer and executing specific time-consuming business logic. Here we use WorkHandler, which supports multiple consumer threads working together, with each message being processed by only one consumer (work queue mode).
```java
@Component
@Slf4j
public class TeamMessageWorkHandler implements WorkHandler<TeamMessageEvent> {

    @Override
    public void onEvent(TeamMessageEvent event) throws Exception {
        // 1. Extract data from the event
        String payload = event.getMessagePayload();
        WebSocketSession session = event.getSession();
        String teamId = event.getTeamId();
        
        log.info("Thread {} starts asynchronous processing of team [{}] message: {}", 
                 Thread.currentThread().getName(), teamId, payload);

        // 2. Simulate time-consuming business logic (e.g., database persistence, complex business calculations)
        Thread.sleep(100); 

        // 3. Construct the response result and broadcast it to other team members via WebSocket
        String response = "{\"status\":\"success\",\"data\":\"Processing complete\"}";
        if (session != null && session.isOpen()) {
            session.sendMessage(new TextMessage(response));
        }
    }
}
```
1. Configure the Disruptor Instance

Configure Disruptor in Spring Boot. Set the RingBuffer size appropriately and bind the consumer thread pool.
```java
@Configuration
public class DisruptorConfig {

    @Resource
    private TeamMessageWorkHandler teamMessageWorkHandler;

    @Bean("teamMessageDisruptor")
    public Disruptor<TeamMessageEvent> teamMessageDisruptor() {
        // Specify the RingBuffer size, must be a power of 2
        int bufferSize = 1024 * 256;

        // Create the Disruptor
        Disruptor<TeamMessageEvent> disruptor = new Disruptor<>(
                TeamMessageEvent::new,
                bufferSize,
                ThreadFactoryBuilder.create()
                .setNamePrefix("disruptorWorker")
                .build()
        );

        // Set the consumer handler (running in WorkerPool mode, multiple threads collaborate to consume)
        disruptor.handleEventsWithWorkerPool(teamMessageWorkHandler);

        // Start the Disruptor
        disruptor.start();
        return disruptor;
    }
}
```
1. Define the Producer

The producer is responsible for delivering high-concurrency messages received via WebSocket to the Disruptor's RingBuffer. The delivery process is lock-free and extremely fast.
```java
@Component
public class TeamMessageProducer {

    @Resource
    private Disruptor<TeamMessageEvent> teamMessageDisruptor;

    /**
     * Publish a message to the RingBuffer
     */
    public void publishEvent(String payload, WebSocketSession session, String teamId, String userId) {
        RingBuffer<TeamMessageEvent> ringBuffer = teamMessageDisruptor.getRingBuffer();
        
        // 1. Get the next available sequence number
        long next = ringBuffer.next();
        try {
            // 2. Get the pre-allocated Event object corresponding to the sequence number and fill in the properties
            TeamMessageEvent event = ringBuffer.get(next);
            event.setMessagePayload(payload);
            event.setSession(session);
            event.setTeamId(teamId);
            event.setUserId(userId);
        } finally {
            // 3. Publish the event (must be published in finally to ensure the sequence number is submitted correctly even if an exception occurs)
            ringBuffer.publish(next);
        }
    }

    /**
     * Graceful shutdown: wait for all events to be processed before closing
     */
    @PreDestroy
    public void destroy() {
        if (teamMessageDisruptor != null) {
            teamMessageDisruptor.shutdown();
        }
    }
}
```
1. Integrate into the WebSocket Handler

Finally, in the WebSocket network event handler, we no longer process business synchronously. Instead, we directly throw the message to the Producer on the spot, immediately releasing the WebSocket thread to handle the next network request.
```java
@Component
@Slf4j
public class GameTeamWebSocketHandler extends TextWebSocketHandler {

    @Resource
    private TeamMessageProducer teamMessageProducer;

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        // Extract common parameters established during handshake interception from session attributes
        String teamId = (String) session.getAttributes().get("teamId");
        String userId = (String) session.getAttributes().get("userId");
        String payload = message.getPayload();

        // Directly "instantly deliver" high-concurrency network messages to the asynchronous queue, without any time-consuming processing here
        teamMessageProducer.publishEvent(payload, session, teamId, userId);
    }
}
```

## FAQ
### Why does Disruptor need a graceful shutdown?
In a production environment, distributed systems are constantly facing version iterations, service restarts, or scaling down. If we forcefully terminate the process when shutting down the Spring container, it may lead to the following disastrous consequences:

1. Data Loss: At this point, there may still be backlogged team messages in the RingBuffer that have not been processed by consumers. A forceful shutdown will cause this data to disappear.

2. State Inconsistency: The consumer thread may be in a critical state halfway through execution (e.g., the database has just finished writing to table A but has not yet updated table B). Direct interruption can lead to dirty states in business data.

## Summary
By introducing the Disruptor framework, we have successfully achieved full asynchronous decoupling between WebSocket network IO threads and time-consuming business logic threads.

In a high-concurrency environment, WebSocket threads only need to be responsible for the two extremely fast steps of "receiving messages -> inserting into the RingBuffer", reducing the single processing time from hundreds of milliseconds to nanoseconds. The underlying lock-free ring array and memory barrier black technology ensure the safety and high performance of multi-threaded message delivery, perfectly solving the message blocking problem in multi-player team collaboration systems.
