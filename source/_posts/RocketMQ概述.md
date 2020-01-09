---
title: RocketMQ概述
date: 2020-01-09 20:23:31
tags:
---
## 一、什么是RocketMQ？
RocketMQ阿里开源的一款高性能、高吞吐量的分布式消息中间件

**什么是企业级分布式消息中间件？**
简单来说就是升级版的消息队列，而且它要满足如下功能

- 消除单点故障
- 保证消息传输可靠性
- 可应对大流量冲击

## 二、RocketMQ的功能介绍

 1. 应用解耦
 2. 流量削峰
 3. 消息分发

**应用解耦**
以电商应用为例，用户创建订单后，如果搞合调用库存系统 、 物流系统 、 支付系统，任何一个子系统出了故障或者因为升级等原因暂时不可用，都会造成下单操作异常，影响用户使用体验 。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130151351288.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)
当转变成基于消息队列的方式后，系统可用性就高多了，比如物流系统因为发生故障，需要几分钟的时间来修 复 ，在这几分钟的时间里， 物流系统要处理的内容被缓存在消息队列里，用户的下单操作可以正常完成 。 
当物流系统恢复后，补充处理存储在消息队列里的订单信息即可，终端用户感知不到物流系统发生过几分钟的故障 。

**流量削峰**
每年的双十一，淘宝的很多活动都在 0 点的时候开启，大部分应用系统流量会在瞬间猛增，这个时候如果没有缓冲机制，不可能承受住短时大流 量 的冲 击。 
通过利用消息队列，把大量的请求暂存起来，分散到相对长的一段时间内 处理，能大大提高系统的稳定性和用户体验 。

**消息分发**
不同子系统将日志写入消息队列，数据使用方根据各自需求订阅对应的消息即可。
甚至某个团队处理完的结果数据也可以 写人消息队列，作为数据的产生方，供其他团队使用，避免重复计算 。
![在这里插入图片描述](https://img-blog.csdnimg.cn/201901301521021.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)

## 三、RocketMQ的前身今世
- 2010 年， B2B 开始大规模使用 ActiveMQ 作为消息内核，随着阿里业务 的快速发展，急需一款支持顺序消息，拥有海量消息堆积能力的消息中间件， MetaQ 1.0 在 2011 年诞生 。
- 2012年， MetaQ已经发展到了3.0版本，并抽象出了通用的消息引擎 RocketMQ。 随后，对 RocketMQ 进行了开源 ， 阿里的消息中间件正式走人了 公众视野 。
- 2015年， RocketMQ已经经历了多年双十一的洗礼，在可用性、 可靠性以 及稳定性等方面都有出色的表现。
- 2016 年， MetaQ 在双十一期间承载了万亿级消息的流转，跨越了一个新的
里程碑 ，同时 RocketMQ 进入 Apache 孵化 。

## 四、RocketMQ各部分角色概述
RocketMQ的角色划分可以分为四类：

- Producer
- Consumer
- Broker
- NameServer

![在这里插入图片描述](https://img-blog.csdnimg.cn/201901301532523.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)

分布式消息队列是用来高效地传输消息的，它的功能和现实生活中的邮局收发信件很类似，我们可以类比地说一下相应的模块 。 
现实生活中的邮政系统要正常运行，离不开这四个角色， 
- 发信者（Producer）
- 收信者（Consumer）
- 负责暂存 、 传输的邮局（Broker）
- 负责协调各个地方邮局的管理机构 （NameServer）

启动 RocketMQ 的顺序是先启动 NameServer，再启动 Broker，这时候消息队列已 经可以提供服务了，想发送消息就使用 Producer来发送，想接收消息就使用 Consumer来接收 。 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130153445870.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)
如上图所示，再复杂一些，很多应用程序既要发送，又要接收，可以启动多个Producer 和 Consumer 来发送多种消息，同时接收多种消息 。
同时为了消除单点故障，增加可靠性或增大吞吐量 ，可以在多台机器上部署多个 NameServer 和 Broker，为每个Broker部署一个或多个Slave。

**名词补充：**
- **Topic:**
一个分布式消息队列中间件部署好以后，可以给很多个业务提供服务，同一个业务也有不同类型的消息要投递 ， 这些不同类型的消息通过不同的 Topic 名称来区分 。 
- **Message Queue:** 
如果一个 Topic要发送和接收的数据量非常大，需要能支持增加并行处理的机器来提高处理速度，这时候一个Topic 可以根据需求设置一个或多个Message Queue（ Message Queue类似分区或 Partition）。
Topic有了多个Message Queue后，消息可以并行地向各个Message Queue 发送，消费者也可以并行地从多个 Message Queue读取消息并消费。
