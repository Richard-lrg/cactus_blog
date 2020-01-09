---
title: RocketMQ角色详解之NameServer
date: 2020-01-09 20:34:51
tags:
---
### 一、NameServer的功能
**NameServer是整个消息队列中的状态服务器，集群的各个组件通过它来了解全局的信息 。** 

需要了解的两个知识：

 - 热备份： NamServer可以部署多个，相互之间独立，其他角色同时向多个NameServer 机器上报状态信息。
 - 心跳机制： NameServer 中的 Broker、 Topic等状态信息不会持久存储，都是由各个角色定时上报并存储到内存中，超时不上报的话， NameServer会认为某个机器出故障不可用了，其他组件会把这个机器从可用列表里移除 。 （每 10秒检查一次，时间戳超过 2分钟则认为Broker已失效。)

### 二、NameServer的集群状态存储结构
**集群的状态就保存于五个变量中，NameServer 的主要工作就是维护这五个变量中存储的信息。**

**1.private final HashMap<String， List<QueueData>> topicQueueTable**

```
 Key 是 Topic 的名称，它存储了所有Topic 的属性信息 。
Value 是个 QueueData 队列,队里的长度等于这个 Topic 数据存储的 MasterBroker的个数。
QueueData里存储着 Broker的名称、 读写queue的数量、 同步标识等。
```

**2.private final HashMap<String， BrokerData> Broker- AddrTable**

```
这个结构存储着一个 BrokerName 对应的属性信 息，
包括所属的 Cluster 名称，Master Broker 和多个 Slave Broker 的地址信息 。
```

**3.private final HashMap<String， Set<String>> ClusterAddrTable**

```
存储的是集群中 Cluster 的信息
Cluster 名称对应一个由 BrokerName组成的集合
```

**4.private final HashMap<String， BrokerLivelnfo> Broker- LiveTable**

```
BrokerLiveTable 存储的内容是这台 Broker机器的实时状态，
包括上次更新状态的时间戳， NameServer会定期检查这个时间戳，超时没有更新就认为这个 Broker无效了，将其从 Broker列表里清除。
```

**5.private fina l HashMap<String ， List<String>> filterServerTable**

```
Filter Server是过滤服务器，是 RocketMQ 的一种服务端过滤方式。
一个Broker可以有一个或多个Filter Server。 
Key 是 Broker 的地址
Value 是和这个 Broker关联的多个 Filter Server 的地址 。
```

### 三、为什么不用已有的Zookeeper？

ZooKeeper 的功能很强大，包括自动 Master 选举等， RocketMQ 的架构设计决定了它不需要进行 Master选举，用不到这些复杂的功能，只需要一个轻量级的元数据服务器就足够了 。
中间件对稳定性要求很高， RocketMQ的 NameServer只有很少的代码，容易维护，所以不需要再依赖另一个中间件，从而减少整体维护成本 。
