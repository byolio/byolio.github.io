---
layout:     post
title:      "RocketMQ的NameSrv的工作原理"
subtitle:   "RocketMQ的NameSrv原理解释"
date:       2024-12-7
author:     "Byolio"
header-img: "img/RocketMQNameSrv.jpg"
catalog: true
tags:
    - RocketMQ
---
> 本文旨在介绍RocketMQ的NameSrv的工作原理。

## 什么是NameSrv
NameSrv是RocketMQ的一个重要组件，它像一个看板一样负责管理和维护RocketMQ的元数据信息，包括Broker的注册、路由信息的维护等。NameSrv提供了一个集中式的服务，用于存储和管理RocketMQ的配置信息和元数据，以便其他组件能够通过NameSrv获取到这些信息, 因此其在Broker启动前需要先启动。

## NameSrv与Broker
Broker启动时，会每30s向NameSrv上报自己的信息，NameSrv每10s钟会扫描记录的broker的信息，剔除掉超过120s没有上报的broker。 \
其上报信息包括:
1. topicQueueTable
其描述了每个topic上的queueData信息,  包含Broker的名字，以及对应的队列数量等等。
2. brokerAddrTable
其描述了每个brokerName的信息，包含Broker的名字，以及对应的ip地址等等。
3. clusterAddrTable
Broker有集群的概念，这个map就是记录集群和对应Broker的关联关系, 通过集群名就能找到所有在这个集群下的`Broker`。
4. brokerLiveTable
这个map记录了每个Broker的存活状态，通过BrokerAddr就能找到对应的Broker最新上报的更新时间、版本号、读写channel等。
5. filterServerTable
这个map记录了每个Broker的过滤服务的地址，通过BrokerAddr就能找到对应的过滤服务的地址。

## NameSrv对Producer和Consumer的作用
Broker 上报给 NameSrv 的信息，就是为了给 Producer 和 Consumer 使用的。\
Producer 每30s从 NameSrv 获取有关 Topic 的路由信息，然后跟对应的 Broker 建立长连接, 并缓存路由信息在内存里(重启会丢失缓存)，后面直接将消息发送给对应的 Topic。\
Consumer 在启动的时候需要连接 NamerSrv，跟它建立长连接，并且从它获取 topic 和 broker 的映射关系, 并和有关系的 Broker 建立长连接，然后订阅对应的 Topic，当有消息发送过来的时候，会推送给 Consumer。

## 如何应对NameSrv故障
为了应对NameSrv故障, 需要部署多台 NameSrv 形成 NameSrv 集群 。 \
因为 Rocket 实现的 NameSrv 非常轻量级, 集群之间的 NameSrv 不会进行任何的信息交互, 因此 Broker 需要给集群内的每一台 NameSrv 都上报路由信息，这样每台 NameSrv 存储的都是完整的路由信息。 \
Producer 和 Consumer 则因为多台 NameSrv 的数据都是一样的，因此它们只需要随机选择集群内的一台 NameSrv 进行长连接即可获取全量的路由信息。
