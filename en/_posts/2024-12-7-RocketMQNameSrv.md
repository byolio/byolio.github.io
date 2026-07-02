---
layout: post
title: How NameSrv Works in RocketMQ
subtitle: RocketMQ的NameSrv原理解释
date: 2024-12-7
author: Byolio
header-img: img/RocketMQNameSrv.jpg
catalog: true
tags:
  - RocketMQ
lang: en
---

> 🌐 [中文](/2024-12-7-RocketMQNameSrv/) | [English](/en/en/_posts/2024-12-7-RocketMQNameSrv/)

> This article aims to introduce how NameSrv works in RocketMQ.

## What is NameSrv
NameSrv is an important component of RocketMQ. It acts like a dashboard, responsible for managing and maintaining RocketMQ's metadata information, including Broker registration, routing information maintenance, etc. NameSrv provides a centralized service for storing and managing RocketMQ's configuration information and metadata, so that other components can obtain this information through NameSrv. Therefore, it needs to be started before the Broker starts.

## NameSrv and Broker
When a Broker starts, it reports its own information to NameSrv every 30 seconds. NameSrv scans the recorded Broker information every 10 seconds and removes Brokers that have not reported for more than 120 seconds. \
The reported information includes:
1. topicQueueTable
   It describes the queueData information for each topic, including the Broker name and the corresponding number of queues, etc.
2. brokerAddrTable
   It describes the information for each brokerName, including the Broker name and the corresponding IP address, etc.
3. clusterAddrTable
   Brokers have the concept of clusters. This map records the association between clusters and corresponding Brokers. By using the cluster name, you can find all Brokers under that cluster.
4. brokerLiveTable
   This map records the live status of each Broker. By using the BrokerAddr, you can find the latest reported update time, version number, read/write channel, etc., of the corresponding Broker.
5. filterServerTable
   This map records the address of the filter service for each Broker. By using the BrokerAddr, you can find the corresponding filter service address.

## The Role of NameSrv for Producer and Consumer
The information reported by Brokers to NameSrv is intended for use by Producers and Consumers. \
Producers obtain routing information about Topics from NameSrv every 30 seconds, then establish a long connection with the corresponding Broker, cache the routing information in memory (which will be lost upon restart), and subsequently send messages directly to the corresponding Topic. \
When a Consumer starts, it needs to connect to NameSrv, establish a long connection with it, obtain the mapping between topics and brokers, establish long connections with the relevant Brokers, and then subscribe to the corresponding Topic. When a message is sent, it will be pushed to the Consumer.

## How to Handle NameSrv Failures
To handle NameSrv failures, multiple NameSrv instances need to be deployed to form a NameSrv cluster. \
Because RocketMQ's NameSrv implementation is very lightweight, NameSrv instances in the cluster do not exchange any information with each other. Therefore, the Broker needs to report routing information to every NameSrv in the cluster, so that each NameSrv stores complete routing information. \
Since the data on multiple NameSrv instances is the same, Producers and Consumers only need to randomly select one NameSrv in the cluster to establish a long connection and obtain the full routing information.
