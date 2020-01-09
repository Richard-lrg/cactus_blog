---
title: RocketMQ使用场景之吞吐量优先Case
date: 2020-01-09 20:36:45
tags:
---

### 一、Broker端进行消息过滤，提高吞吐量
**在 Broker端进行消息过滤，可以减少无效消息发送到 Consumer，少占用网络带宽从而提高吞吐量 。**

##### 过滤方式：
**方式1：**
通过tag 和 key 进行过滤(在创建Message时设置）
Tag和 Key的主要差别是使用场景不同 
Tag用在 Consumer的代码中，用来进行服务端消息过滤
Key 主要用于通过命令行查询消息 。

**方式2：**
通过sql表达式的方式进行过滤

SqlProducer.java
```java
package org.apache.rocketmq.example.filter;

import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

public class SqlProducer {

    public static void main(String[] args) {
        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
        try {
            producer.start();
        } catch (MQClientException e) {
            e.printStackTrace();
            return;
        }

        for (int i = 0; i < 10; i++) {
            try {
                String tag;
                int div = i % 3;
                if (div == 0) {
                    tag = "TagA";
                } else if (div == 1) {
                    tag = "TagB";
                } else {
                    tag = "TagC";
                }
                Message msg = new Message("TopicTest",
                    tag,
                    ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET)
                );
                msg.putUserProperty("a", String.valueOf(i));

                SendResult sendResult = producer.send(msg);
                System.out.printf("%s%n", sendResult);
            } catch (Exception e) {
                e.printStackTrace();
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e1) {
                    e1.printStackTrace();
                }
            }
        }
        producer.shutdown();
    }
}
```

SqlConsumer.java
```java
package org.apache.rocketmq.example.filter;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.MessageSelector;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

public class SqlConsumer {

    public static void main(String[] args) {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name_4");
        try {
            consumer.subscribe("TopicTest",
                MessageSelector.bySql("(TAGS is not null and TAGS in ('TagA', 'TagB'))" +
                    "and (a is not null and a between 0  3)"));
        } catch (MQClientException e) {
            e.printStackTrace();
            return;
        }

        consumer.registerMessageListener(new MessageListenerConcurrently() {

            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                ConsumeConcurrentlyContext context) {
                System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), msgs);
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        try {
            consumer.start();
        } catch (MQClientException e) {
            e.printStackTrace();
            return;
        }
        System.out.printf("Consumer Started.%n");
    }
}

```

**方式3：**
Filter Server方式过滤

producer.java

```java
package org.apache.rocketmq.example.filter;

import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

public class Producer {
    public static void main(String[] args) throws MQClientException, InterruptedException {
        DefaultMQProducer producer = new DefaultMQProducer("ProducerGroupName");
        producer.start();

        try {
            for (int i = 0; i < 6000000; i++) {
                Message msg = new Message("TopicFilter7",
                    "TagA",
                    "OrderID001",
                    "Hello world".getBytes(RemotingHelper.DEFAULT_CHARSET));

                msg.putUserProperty("SequenceId", String.valueOf(i));
                SendResult sendResult = producer.send(msg);
                System.out.printf("%s%n", sendResult);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        producer.shutdown();
    }
}

```
consumer.java

```java
package org.apache.rocketmq.example.filter;

import java.io.File;
import java.io.IOException;
import java.util.List;
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.MixAll;
import org.apache.rocketmq.common.message.MessageExt;

public class Consumer {

    public static void main(String[] args) throws InterruptedException, MQClientException, IOException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ConsumerGroupNamecc4");

        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
        File classFile = new File(classLoader.getResource("MessageFilterImpl.java").getFile());

        String filterCode = MixAll.file2String(classFile);
        consumer.subscribe("TopicTest", "org.apache.rocketmq.example.filter.MessageFilterImpl",
            filterCode);

        consumer.registerMessageListener(new MessageListenerConcurrently() {

            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                ConsumeConcurrentlyContext context) {
                System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), msgs);
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        consumer.start();

        System.out.printf("Consumer Started.%n");
    }
}
```


### 二、提高Consumer处理能力
**1.提高消费并行度**
方式1：增加consumer实例
方式2：提高单个 Consumer 实例中的并行处理的线程数
（修改 consumeThreadMin 和 consumeThreadMax)

**2.批量方式进行消费**
设置 Consumer 的 consumeMessageBatchMaxSize 
这个参数 ，默认是 1，如果设置为 N，在消息多的时候每次收到的是个长度为 N的消息链表

**注意：**
broker配置文件的maxTransferCountOnMessageInMemory参数也要相应增加
该参数指的是服务器但允许在内存中传递的最大消息数，默认是32条

**3.检测延时情况，跳过非重要消息**
Consumer 在消费的过程中， 如果发现由于某种原因发生严重的消息堆积，短时间无法消除堆积
这个时候可以选择丢弃不重要 的消息，使 Consumer尽快追上 Producer 的进度

### 三、Consumer负载均衡
**DefaultMQPushConsumer的负载均衡：**
负载均衡过程不需要使用者操心，客户端程序会自动处理
每启动一个consumer就会触发一次doRebalance,ConsumerGroup加入新consuemr时，也会触发doRebalace

**注意：**
负载均衡算法默认使用AllocateMessageQueueAveragely。(分配粒度只到 Message Queue)
负载均衡的结果与 Topic 的 Message Queue 数量，以及 ConsumerGroup 里的 Consumer 的数量有关 。
3m2c: 2 1
3m4c: 1 1 1 0
可见Message Queue数量设置过小不利于做负载均衡,通常情况下，应把一个 Topic 的Message Queue 数设置为 16。

**DefaultMQPullConsumer 的负载均衡：**
**1.通过registerMessageQueueListener 函数**
registerMessageQueueListener函数在有新的Consumer加入或退出时被触发。 

**2.通过MQPullConsumerScheduleService类**
具体参考代码如下：

```java
package org.apache.rocketmq.example.simple;

import org.apache.rocketmq.client.consumer.MQPullConsumer;
import org.apache.rocketmq.client.consumer.MQPullConsumerScheduleService;
import org.apache.rocketmq.client.consumer.PullResult;
import org.apache.rocketmq.client.consumer.PullTaskCallback;
import org.apache.rocketmq.client.consumer.PullTaskContext;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.message.MessageQueue;
import org.apache.rocketmq.common.protocol.heartbeat.MessageModel;

public class PullScheduleService {

    public static void main(String[] args) throws MQClientException {
        final MQPullConsumerScheduleService scheduleService = new MQPullConsumerScheduleService("GroupName1");

        scheduleService.setMessageModel(MessageModel.CLUSTERING);
        scheduleService.registerPullTaskCallback("TopicTest", new PullTaskCallback() {

            @Override
            public void doPullTask(MessageQueue mq, PullTaskContext context) {
                MQPullConsumer consumer = context.getPullConsumer();
                try {

                    long offset = consumer.fetchConsumeOffset(mq, false);
                    if (offset < 0)
                        offset = 0;

                    PullResult pullResult = consumer.pull(mq, "*", offset, 32);
                    System.out.printf("%s%n", offset + "\t" + mq + "\t" + pullResult);
                    switch (pullResult.getPullStatus()) {
                        case FOUND:
                            break;
                        case NO_MATCHED_MSG:
                            break;
                        case NO_NEW_MSG:
                        case OFFSET_ILLEGAL:
                            break;
                        default:
                            break;
                    }
                    consumer.updateConsumeOffset(mq, pullResult.getNextBeginOffset());

                    context.setPullNextDelayTimeMillis(100);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });

        scheduleService.start();
    }
}
```

### 四、提高Producer发送速度
**途径1：**
增加Producer并发量，使用多个Producer实例同时发送

**注意：**
不用担心多 Producer 同时写会降低消息写磁盘的效率， RocketMQ 引入了 一个并发窗口，在窗口内消息可以并发地写人 DirectMem 中 ， 然后异步地将连续一段无空洞的数据刷入文件系统当中 。

**途径2：**
可靠性要求不高的场景下，可以采用OneWay方式发送。
单向（Oneway）发送特点为发送方只负责发送消息，不等待服务器回应且没有回调函数触发，
即只发送请求不等待应答。 此方式发送消息的过程耗时非常短，一般在微秒级别。

具体参考代码如下：

```java
public class OnewayProducer {
    public static void main(String[] args) throws Exception{
        //Instantiate with a producer group name.
        DefaultMQProducer producer = new DefaultMQProducer("example_group_name");
        //Launch the instance.
        producer.start();
        for (int i = 0; i < 100; i++) {
            //Create a message instance, specifying topic, tag and message body.
            Message msg = new Message("TopicTest" /* Topic */,
                    "TagA" /* Tag */,
                    ("Hello RocketMQ " +
                            i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */
            );
            //Call send message to deliver message to one of brokers.
            producer.sendOneway(msg);

        }
        //Shut down once the producer instance is not longer in use.
        producer.shutdown();
    }
}
```
