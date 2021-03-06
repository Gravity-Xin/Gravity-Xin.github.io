---
title: MQ-NSQ与Kafka的对比
date: 2018-11-15 17:39:51
categories: 
- 系统架构
tags: 
- NSQ
- Kafka
- MQ
---

## 如何让一个topic的消息，能够被多个消费者消费

- nsq：通过引入channel
- kafka：通过引入消费者组

## 如何让mq、生产者和消费者能够自动发现对方

- nsq：通过nsqlookupd提供注册和发现
- kafka：使用zk提供服务注册和发现

## 如何实现集群

- nsq：建议每一个生产者配置一个nsqd
- kafka：
  - Broker：消息中间件处理节点（服务器），一个节点就是一个broker，一个Kafka集群由一个或多个broker组成
  - Topic：kafka对消息进行归类，发送到集群的每一条消息都要指定一个topic
  - Partition：物理上的概念，每个topic包含一个或多个partition，一个partition对应一个文件夹，这个文件夹下存储partition的数据和索引文件，每个partition内部是有序的
  - Producer：生产者，负责发布消息到broker
  - Consumer：消费者，从broker读取消息
  - ConsumerGroup：每个consumer属于一个特定的consumer group，可为每个consumer指定group name，若不指定，则属于默认的group，一条消息可以发送到不同的consumer group，但一个consumer group中只能有一个consumer能消费这条消息

![image](/images/Kafka之基本结构.jpg)

## 消息存储内存/磁盘

- nsq：内存，当队列里消息的数量超过`--mem-queue-size`配置的限制时，才会对消息进行持久化
- kafka：磁盘

## 消费者如何获取消息Push/Pull

- nsq：nsqd push，保证实时性，但是不好进行流量控制
- kafka：consumer pull，消费者可以自主控制，但是延时大，消费者可能pull空，导致效率低

## 数据备份

- nsq：无
- kafka：通过partition机制，对消息进行备份

## 消息有序性

- nsq：消息无序
- kafka：支持顺序消费 -> 要被顺序消费的消息必须放到一个partition中且partition只能被消费者组里面的一个消费者消费

## 消息投递语义

- nsq：至少投递一次
- kafka：支持准确投递一次