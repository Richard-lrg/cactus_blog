---
title: RocketMQ角色详解之Broker
date: 2020-01-09 20:35:23
tags:
---
### 一、Broker概述
**Broker是 RocketMQ 的核心，大部分‘重量级”工作都是由 Broker完成的。
包括接收 Producer 发过来的消息、处理 Consumer 的消费消息请求、消息的持 久化存储、消息的 HA 机制以及服务端过滤功能等 。**


### 二、消息的存储与转发
分布式队列因为有高可靠性的要求，所以数据要通过磁盘进行持久化存储 。
磁盘顺序写速度可以达到 600MB/s，但是随机写的速度只有大概 lOOKB/s。
RocketMQ 充分利用了这个特性，提高消息存盘和网络发送的速度 。


### 三、消息存储结构
**1.RocketMQ消息的存储形式：ConsumeQueue + CommitLog**

- CommitLog：
消息真正的物理存储文件，每台 Broker上的 CommitLog被本机器所有 ConsumeQueue 共享。

- ConsumeQueue:
消息的逻辑队列，类似数据库的索引文件，存储的是指向物理存储的地址。每个 Topic下的每个 Message Queue都有一个对应的 ConsumeQueue文件。

在 CommitLog 中，一个消息的存储长度是不固定的，RocketMQ 采取一些机制，尽量向 CommitLog 中顺序写，但是随机读，每次读取消息队列先读取consumer Queue,然后再通过consumerQueue去commitLog中拿到消息主体。
(ConsumeQueue 的内容也会被写到磁盘里作持久存储 )

**2.此存储结构下的优势**

 1. CommitLog顺序写，大大提高写入效率
 2. 虽然是随机读，但是利用操作系统的 pagecache 机制，可以批量地从磁盘读取作为 cache存到内存中，加速后续的读取速度。
 3. 为了保证完全的顺序写，需要 ConsumeQueue 这个中间结构，在实际情况中，大部分的 ConsumeQueue 能够被全部读入内存，所以这个中间结构的操作速度很快，可以认为是内存读取的速度 。


### 四、高可用机制
**RocketMQ 分布式集群是通过 Master 和 Slave 的配合达到高可用性的**

**1.消费端的高可用：**
Master角色的 Broker支持读和写， Slave角色的 Broker仅支持读
当 Master 不可用或者繁忙的时候， Consumer 会被自动切换到从 Slave 读 。
当一个 Master 角色的机器出现故障后， Consumer 仍然可以从 Slave 读，不影响 Consumer 程序 。 

**2.发送端的高可用：**
在创建 Topic 的时候，把 Topic 的多个 Message Queue创建在多个 Broker组上
(相同 Broker名称，不同 brokerId 的 机器组成 一 个 Broker 组)，
这样当一个Broker 组的 Master 不可用后，其他组的 Master 仍然可用， Producer 仍然可以发送消息 。

### 五、同步刷盘和异步刷盘
**同步刷盘方式 :** 
在返回写成功状态时，消息已经被写入磁盘 。 消息写入内存的 PAGECACHE 后，立刻通知刷盘线程刷盘，然后等待刷盘完成，刷盘线程执行完成后唤醒等待的线程，返回消息写成功的状态 。

**异步刷盘方式 :** 
在返回写成功状态时 ，消息可能只是被写入了内存的 PAGECACHE，写操作的返回快，吞吐量大 ;
当内存里的消息量积累到一定程度时，统一触发 写磁盘动作，快速写人 。

**注意：**
刷盘方式通过 Broker 配置文件里的 flushDiskType 参数进行设置：
SYNC_FLUSH 同步刷盘 
ASYNC_FLUSH 异步刷盘

### 六、同步复制和异步复制
如果一个 Broker组有 Master和 Slave, 消息需要从 Master复制到 Slave上。

**同步复制方式：** 
Master 和 Slave 均写成功 后才反馈给客户端写成功状态

**异步复制方式：**
只要 Master 写成功即可反馈给客户端写成功状态 。

**注意：**
1.复制方式是通过 Broker 配置文件里的 brokerRole 参数进行设置：
ASYNC MASTER 异步复制 
SYNC MASTER 同步复制
SLAVE 无影响

2.通常情况下，应该把
刷盘方式配置成 ASYNC_FLUSH
主从复制方式配置成 SYNC_MASTER 
这样即使有一台机器出故障，仍然能保证数据不丢。
