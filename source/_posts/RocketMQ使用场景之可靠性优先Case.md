---
title: RocketMQ使用场景之可靠性优先Case
date: 2020-01-09 20:36:15
tags:
---
### 一、顺序消息的实现
**1.全局顺序消息**
要保证全局顺序消息，需要先把 Topic 的读写队列数设置为 1，
然后 Producer 和 Consumer 的并发设置也要是1。 
在发送端，要做到 把同一业务 ID 的消息发送到同一个 Message Queue 
（使用 MessageQueueSelector类）

**2.部分顺序消息**
在消费过程中，要做到从 同一个 Message Queue 读取的消息不被并发处理
(使用 MessageListenerOrderly类)
具体实现方式：

```java
在 MessageListenerOrderly 的实现中，为每个 Consumer Queue 加个锁,
消费每个消息前，需要先获得这个消息对应的 Consumer Queue 所对应的锁，
这样保证了同一时间，同一个 Consumer Queue 的消息不被并发消 费，
但不同 Consumer Queue 的消息可以并发处理 。
```
具体参考代码如下：

```java
package org.apache.rocketmq.example.ordermessage;

import java.util.List;
import java.util.concurrent.atomic.AtomicLong;
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeOrderlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeOrderlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerOrderly;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.consumer.ConsumeFromWhere;
import org.apache.rocketmq.common.message.MessageExt;

public class Consumer {

    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name_3");

        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

        consumer.subscribe("TopicTest", "TagA || TagC || TagD");

        consumer.registerMessageListener(new MessageListenerOrderly() {
            AtomicLong consumeTimes = new AtomicLong(0);

            @Override
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
                context.setAutoCommit(false);
                System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), msgs);
                this.consumeTimes.incrementAndGet();
                if ((this.consumeTimes.get() % 2) == 0) {
                    return ConsumeOrderlyStatus.SUCCESS;
                } else if ((this.consumeTimes.get() % 3) == 0) {
                    return ConsumeOrderlyStatus.ROLLBACK;
                } else if ((this.consumeTimes.get() % 4) == 0) {
                    return ConsumeOrderlyStatus.COMMIT;
                } else if ((this.consumeTimes.get() % 5) == 0) {
                    context.setSuspendCurrentQueueTimeMillis(3000);
                    return ConsumeOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT;
                }

                return ConsumeOrderlyStatus.SUCCESS;
            }
        });

        consumer.start();
        System.out.printf("Consumer Started.%n");
    }

}
```

### 二、消息重复问题
消息重复一般情况下不会发生，但是如果消息量大，网络有波动，消息重复就是个大概率事件。 主要造成的原因是： 
Producer有个setRetryTimesWhenSendFailed函数, 设置在同步方式下自动重试的次数，默认值是 2，这样当第一次发送消息时， Broker端接收到了消息但是没有正确返回发送成功的状态，就造成了消息重复。

**解决办法：**

 1. 保证消费逻辑的幕等性(多次调用和一次调用效果相同)
 2. 维护一个巳消费消息的记录，消费前查询这个消息是否被消费过 （只能在入库时，手动写代码去重）


### 三、消息优先级
**RocketMQ 是个先入先出的队列，不支持消息级别或者 Topic 级别的优先级 ，只能通过间接方式。**

**情景1：**
同一Topic下某一消息需要被及时处理，可另外开辟topic
**情景2：**

```java
一个订单处理系统，接收从 100家快递门店过来的请求，把这些请求通过 Producer 写人 RocketMQ ;
订单处理程序通过 Consumer 从队列里读取消 息并处理，每天最多处理 1 万单 。 
如果这 100 个快递门店中某几个门店订单量大增，
比如门店一接了个大客户，一个上午就发出 2万单消息请求，
这样其他的 99 家门店可能被迫等待门店一的 2 万单处理完，
也就是两天后订单才能被处理，显然很不公平 。
```
解决：
```java
创建一个 Topic， 设置 Topic 的 MessageQueue 数 量 超过 100 个，
Producer根据订单的门店号，把每个门店的订单写人 一 个 MessageQueue。 
DefaultMQPushConsumer默认是采用循环的方式逐个读取一个 Topic 的 所有 
MessageQueue，这样如果某家门店订单 量 大增，这家门店对应的 
MessageQueue 消息数增多，等待时间增长，但不会造成其他家门店等待时间增长。
```

**情景3：**

```java
一个应用程序同时处理 TypeA、 TypeB、 TypeC 三类消息 。 
TypeA 处于第 一优先级，要确保只要有 TypeA消息，必须优先处理; 
TypeB处于第二优先级; 
TypeC 处于第三 优先级 。 
```
解决：

```java
对这种要求，或者逻辑更复杂的要求，需要自己编码实现优先级控制，
如果上述的三类消息在一个 Topic 里，可以使 用 PullConsumer，
自主控制 Messag巳Queue 的遍历，以及消息的读取;
```

### 四、各种故障对消息队列的影响
故障清单：

```
1) Broker正常关闭，启动;
2) Broker异常 Crash，然后启动;
3) OS Crash，重启;
4 )机器断电，但能马上恢复供电;
5 )磁盘损坏;
6) CPU、 主板、内存等关键设备损坏 。
```
分析：

```java
1 ）情况 属于可控的软件 问题，内存中的数据不会丢失 。 
若重启过程中有持续运行的 Consumer, Master机器出故障后， 
Consumer会自动重连到对应的 Slave 机器，不会有消息丢失和偏差 。 
当 Master 角色的机器 重启 以 后， Consumer又会重新连接到 Master机器
若有持续运行的 Producer，一台Master 出故障后，
Producer只能向 Topic下其他的 Master机器发送消息，
如果Producer采用同步发送方式，不会有消息丢失 。

2、3、4）情况属于软件故障，内存的数据可能丢失，所以刷盘策略不同，造成的影响也不同，
如果 Master、 Slave都配置成 SYNC_FLUSH，可以达到和第 1 种情况相同的效果 。

5、6 ）情况属于硬件故障 ，发生第 5、6 种情况的故障，原有机器的磁盘数据可能会丢失。 如果Master和Slave机器间配置成同步复制方式，某一台机器发生 5 或 6 的故障，也可以达到消息不丢失的效果 。 机器间是异步复制，两次 Sync间的消息会丢失。
```

常规思路：

```java
总的来说，当设置成:
1 )多 Master，每个 Master 带有 Slave; 
2 )主从之间设置成 SYNC_MASTER; 
3 ) Producer 用同步方式写;
4 )刷盘策略设置成 SYNC FLUSH。
就可以消除单点依赖，即使某台机器出现极端故障也不会丢消息 。
```
