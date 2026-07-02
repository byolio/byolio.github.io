---
layout:     post
title:      "Disruptor框架简介和使用"
subtitle:   "Disruptor框架解决高并发处理团队消息问题"
date:       2026-6-18
author:     "Byolio"
header-img: "img/disruptor.png"
catalog: true
tags:
    - java
    - Disruptor
    - JUC
    - SpringBoot
---

> 🌐 [中文](https://www.byolio.blog/2026-6-18-Disruptor/) | [English](https://www.byolio.blog/en/en/_posts/2026-6-18-Disruptor/)

## 引言
在使用 WebSocket 编写多人团队协作或实时聊天系统时，通常会遇到一个性能瓶颈：当 WebSocket 线程接收到一个团队消息后，如果直接在当前线程中同步处理耗时的业务逻辑（如数据库持久化、权限校验、广播分发等），就会导致该线程被长时间占用。

在高并发场景下，这种同步处理方式会迅速耗尽网络线程池，导致后续的其他团队消息被阻塞，甚至引发连接超时和消息丢失，系统整体吞吐量断崖式下跌。

为了解决这个问题，我们需要引入**异步解耦**机制。而在众多队列方案中，**LMAX Disruptor** 框架凭借其极致的性能，成为了我们实现高效、低延迟团队消息处理的首选利器。

## Disruptor简介
Disruptor 是由英国外汇交易公司 LMAX 开发的一个高性能、低延迟的**进程内（In-Memory）并发框架**。它最初的设计目的是为了解决金融交易系统在多线程环境下的低延迟和高吞吐量需求。

在传统的 Java 并发编程中，我们通常使用 `ArrayBlockingQueue` 或 `LinkedBlockingQueue` 来实现生产者-消费者模型。然而，Disruptor 另辟蹊径，抛弃了传统的队列结构，在几乎不使用显式锁(Lock-Free)的情况下，实现了每秒处理数百万条订单的恐怖性能。

### 1. 传统队列的性能痛点
在了解 Disruptor 为什么快之前，我们需要知道传统并发队列（如 `BlockingQueue`）在高并发下的三大痛点：
1. **严重的锁竞争**：传统的阻塞队列在入队（Put）和出队（Take）时，通常需要依赖 `ReentrantLock` 或 `synchronized` 这种重型锁来保证线程安全。高并发下，线程上下文切换（Context Switch）的开销极大。
2. **伪共享（False Sharing）问题**：传统的链表或数组结构中，队列的头尾指针（Head / Tail）在内存中往往挨得很近。多核 CPU 在读取时会将其加载到同一个缓存行（Cache Line）中，导致多个核心之间频繁发生缓存失效，触发写回主存，严重拖慢 CPU 效率。
3. **垃圾回收（GC）压力**：频繁的入队和出队伴随着大量对象的创建和销毁，容易导致 JVM 频繁触发 GC，从而引发系统卡顿（Stop-The-World）。

### 2. Disruptor 的核心黑科技
Disruptor 为了压榨 CPU 的最后一滴性能，引入了以下核心设计：

#### ① 环形数组结构（RingBuffer）
Disruptor 核心采用了一个固定大小的环形数组 `RingBuffer`。
* **无 GC 压力**：数组在初始化时就会填满空的数据对象（Event）。在实际运行时，生产者只是修改这些已存在对象的方法，而不是创建新对象。这实现了**内存复用**，几乎不产生新 GC。
* **位运算寻址**：通过将数组大小设置为 2 的幂次方（如 1024），寻找下一个可用位置时可以使用位运算（`sequence & (length - 1)`）代替取模运算，效率极高。

#### ② 独创的无锁设计：Sequence
Disruptor 没有采用传统的头尾指针，而是让每个生产者和消费者都维护自己的 `Sequence`（序列号，本质是一个递增的 `long` 值）。
* 线程之间通过比较各自的 `Sequence` 来判断是否有新数据可读或是否有空间可写。
* 内部利用底层的 内存屏障(Memory Barrier) 禁止cpu指令重排 以及 CAS（Compare And Swap）原子化操作代替了传统的显式锁，实现了完全的无锁化（Lock-Free）。

#### ③ 解决伪共享：缓存行填充（Padding）

Disruptor 在核心组件（如 `Sequence`）的底层实现中，人为地在核心变量前后填充了 7 个无用的 `long` 变量。这确保了核心序列号能够**独占一个 64 字节的 CPU 缓存行**，彻底避免了伪共享问题，让多核 CPU 的缓存命中率达到极致。


## 如何在SpringBoot中导入Disruptor框架
在pom.xml中导入disruptor依赖项:
```xml
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.4.2</version>
</dependency>
```
## Disruptor的基本使用示例
下面是一个简单的示例，展示了如何使用 Disruptor 来实现一个生产者-消费者模型：
1. 定义事件（Event）

Event 是 RingBuffer 中流转的数据载体。Disruptor 会在启动时预先实例化这些 Event。
```java
@Data
public class TeamMessageEvent {
    private String messagePayload;     // 消息内容 (JSON串或文本)
    private WebSocketSession session;  // 发送者的WebSocket会话
    private String teamId;             // 团队/房间ID
    private String userId;             // 用户ID
}
```
1. 定义消费者（WorkHandler）

消费者负责从 RingBuffer 中获取数据并执行耗时的具体业务逻辑。这里我们使用 WorkHandler，支持多消费者线程共同消费，一条消息只会被其中一个消费者处理（工作队列模式）。
```java
@Component
@Slf4j
public class TeamMessageWorkHandler implements WorkHandler<TeamMessageEvent> {

    @Override
    public void onEvent(TeamMessageEvent event) throws Exception {
        // 1. 提取事件中的数据
        String payload = event.getMessagePayload();
        WebSocketSession session = event.getSession();
        String teamId = event.getTeamId();
        
        log.info("线程 {} 开始异步处理团队 [{}] 的消息: {}", 
                 Thread.currentThread().getName(), teamId, payload);

        // 2. 模拟耗时业务逻辑（如数据库持久化、复杂的业务计算）
        Thread.sleep(100); 

        // 3. 构造响应结果并通过 WebSocket 广播给团队内其他成员
        String response = "{\"status\":\"success\",\"data\":\"处理完成\"}";
        if (session != null && session.isOpen()) {
            session.sendMessage(new TextMessage(response));
        }
    }
}
```
1. 配置 Disruptor 实例

在 Spring Boot 中配置 Disruptor。合理设置 RingBuffer 大小，并绑定消费者线程池。
```java
@Configuration
public class DisruptorConfig {

    @Resource
    private TeamMessageWorkHandler teamMessageWorkHandler;

    @Bean("teamMessageDisruptor")
    public Disruptor<TeamMessageEvent> teamMessageDisruptor() {
        // 指定 RingBuffer 大小，必须是 2 的幂次方
        int bufferSize = 1024 * 256;

        // 创建 Disruptor
        Disruptor<TeamMessageEvent> disruptor = new Disruptor<>(
                TeamMessageEvent::new,
                bufferSize,
                ThreadFactoryBuilder.create()
                .setNamePrefix("disruptorWorker")
                .build()
        );

        // 设置消费者处理器（以 WorkerPool 模式运行，多个线程协作消费）
        disruptor.handleEventsWithWorkerPool(teamMessageWorkHandler);

        // 启动 Disruptor
        disruptor.start();
        return disruptor;
    }
}
```
1. 定义生产者(Producer)

生产者负责将 WebSocket 接收到的高并发消息投递到 Disruptor 的 RingBuffer 中。投递过程是无锁的，速度极快。
```java
@Component
public class TeamMessageProducer {

    @Resource
    private Disruptor<TeamMessageEvent> teamMessageDisruptor;

    /**
     * 将消息发布到 RingBuffer 中
     */
    public void publishEvent(String payload, WebSocketSession session, String teamId, String userId) {
        RingBuffer<TeamMessageEvent> ringBuffer = teamMessageDisruptor.getRingBuffer();
        
        // 1. 获取下一个可用的序列号 (Sequence)
        long next = ringBuffer.next();
        try {
            // 2. 获取该序列号对应的预分配 Event 对象并填充属性
            TeamMessageEvent event = ringBuffer.get(next);
            event.setMessagePayload(payload);
            event.setSession(session);
            event.setTeamId(teamId);
            event.setUserId(userId);
        } finally {
            // 3. 发布事件 (必须在 finally 中发布，确保发生异常时序列号也能正常提交)
            ringBuffer.publish(next);
        }
    }

    /**
     * 优雅停机：等待所有事件处理完成后再关闭
     */
    @PreDestroy
    public void destroy() {
        if (teamMessageDisruptor != null) {
            teamMessageDisruptor.shutdown();
        }
    }
}
```
1. 在 WebSocket 处理器中接入

最后，在 WebSocket 的网络事件处理器中，我们不再同步处理业务，而是直接当场将消息丢给 Producer，随即便释放 WebSocket 线程，让其立刻去接待下一个网络请求。
```java
@Component
@Slf4j
public class GameTeamWebSocketHandler extends TextWebSocketHandler {

    @Resource
    private TeamMessageProducer teamMessageProducer;

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        // 从 session 属性中提取握手时拦截建立的通用参数
        String teamId = (String) session.getAttributes().get("teamId");
        String userId = (String) session.getAttributes().get("userId");
        String payload = message.getPayload();

        // 将高并发的网络消息直接“秒投递”到异步队列中，不在此处做任何耗时处理
        teamMessageProducer.publishEvent(payload, session, teamId, userId);
    }
}
```

## FAQ
### 为什么 Disruptor 需要优雅停机？
在生产环境中，分布式系统随时面临着版本迭代、服务重启或缩容。如果我们在关闭 Spring 容器时直接暴力切断进程，可能会导致以下灾难性后果：

1. 数据丢失：此时 RingBuffer 中可能还有积压的、未被消费者处理完的团队消息，暴力关闭会导致这部分数据凭空蒸发。

2. 状态不一致：消费者线程可能正处于执行到一半的临界状态（例如数据库刚写完 A 表，还没来得及更新 B 表），直接中断会导致业务数据出现脏状态。

## 总结
通过引入 Disruptor 框架，我们成功将 WebSocket 网络IO线程 与 耗时的业务逻辑线程 进行了全异步解耦。

在高并发环境下，WebSocket 线程只需要负责“接收消息 -> 塞入 RingBuffer”这两步极快的操作，单次处理耗时从原本的几百毫秒骤降至纳秒级别。而底层的无锁环形数组和内存屏障黑科技，又确保了多线程投递消息时的安全与高性能，完美化解了多人团队协作系统中的消息阻塞难题。