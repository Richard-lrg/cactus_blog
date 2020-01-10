---
title: RocketMQ角色详解之Consumer
date: 2020-01-09 20:33:45
tags:
---
### 一、常用Consumer类
- DefaultMQPushConsumer
- DefaultMQPullConsumer

**二者的区别：**
DefaultMQPushConsumer：
系统控制读取操作，收到消息后自动调用传入的处理方法来处理，实时性高
DefaultMQPullConsumer：
读取操作中的大部分功能由使用者自主控制 ，灵活性更高。

两种Consumer的选取要主要取决于用户的使用场景，适合的才是最好的。


### 二、DefaultMQPushConsumer的使用
##### 主要参数的设置：
- ConsumerGroupName
- NameServer地址及端口号
- Topic

**ConsumerGroupName**

通过设置ConsumerGroupName将多个 Consumer组织到一起对同一topic进行消费， 提高并发处理能力，且ConsumerGroupName需与消息模式 (MessageModel)配合使用

MessageModel = Clustering时，
在 Clustering模式下，同一个 ConsumerGroup(GroupName相同) 里的每个Consumer 只消费所订阅消息的一部分内容， 同一个ConsumerGroup 里所有的 Consumer消费的内容合起来才是所订阅 Topic 内容的整体， 从而达到负载均衡的目的 。

MessageModel = Broadcasting时，
同一个 ConsumerGroup里的每个 Consumer都 能消费到所订阅 Topic 的全部消息，也就是一个消息会被多次分发，被 多个 Consumer消费。

**NameServer地址及端口号**
可以填写多个 ，用分号隔开，用以消除单点故障
如 “ip1:port;ip2:port;ip3 :port” 

**Topic**
Topic名称用来标识消息类型， 需要提前创建。
如果不需要消费某 个 Topic 下的所有消息，可以通过指定消息的 Tag 进行消息过滤，在填写 Tag 参数的位置，若赋值为 null 或 * 则表示要消费此Topic 的所有消息 。

**参数设置完毕后将MessageListener注册到该consumer即可自动监听处理接收到的消息**
##### 详细代码如下：
```java
package org.apache.rocketmq.example.simple;

import java.util.List;
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.consumer.ConsumeFromWhere;
import org.apache.rocketmq.common.message.MessageExt;

public class PushConsumer {

    public static void main(String[] args) throws InterruptedException, MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("CID_JODIE_1");
        consumer.subscribe("Jodie_topic_1023", "*");
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
        consumer.setConsumeTimestamp("20170422221800");
        consumer.registerMessageListener(new MessageListenerConcurrently() {

            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), msgs);
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
        System.out.printf("Consumer Started.%n");
    }
}
```

### DefaultMQPullConsumer的使用
DefaultMQPullConsumer对消息队列的主要处理流程如下：

 1. 获取Message Queue并遍历
 2. 维护Offsetstore
 3. 根据不同消息状态做不同的处理

具体代码参考如下：
```java
package org.apache.rocketmq.example.simple;

import java.util.HashMap;
import java.util.Map;
import java.util.Set;
import org.apache.rocketmq.client.consumer.DefaultMQPullConsumer;
import org.apache.rocketmq.client.consumer.PullResult;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.message.MessageQueue;

public class PullConsumer {
    private static final Map<MessageQueue, Long> OFFSE_TABLE = new HashMap<MessageQueue, Long>();

    public static void main(String[] args) throws MQClientException {
        DefaultMQPullConsumer consumer = new DefaultMQPullConsumer("please_rename_unique_group_name_5");

        consumer.start();

        Set<MessageQueue> mqs = consumer.fetchSubscribeMessageQueues("TopicTest1");
        for (MessageQueue mq : mqs) {
            System.out.printf("Consume from the queue: %s%n", mq);
            SINGLE_MQ:
            while (true) {
                try {
                    PullResult pullResult =
                        consumer.pullBlockIfNotFound(mq, null, getMessageQueueOffset(mq), 32);
                    System.out.printf("%s%n", pullResult);
                    putMessageQueueOffset(mq, pullResult.getNextBeginOffset());
                    switch (pullResult.getPullStatus()) {
                        case FOUND:
                            break;
                        case NO_MATCHED_MSG:
                            break;
                        case NO_NEW_MSG:
                            break SINGLE_MQ;
                        case OFFSET_ILLEGAL:
                            break;
                        default:
                            break;
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }

        consumer.shutdown();
    }

    private static long getMessageQueueOffset(MessageQueue mq) {
        Long offset = OFFSE_TABLE.get(mq);
        if (offset != null)
            return offset;

        return 0;
    }

    private static void putMessageQueueOffset(MessageQueue mq, long offset) {
        OFFSE_TABLE.put(mq, offset);
    }

}
```
