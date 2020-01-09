---
title: RocketMQ的环境搭建与相关配置
date: 2020-01-09 20:31:25
tags:
---
## 一、RocketMQ的下载
[http://rocketmq.apache.org/dowloading/releases/](http://rocketmq.apache.org/dowloading/releases/)

## 二、RocketMQ环境搭建
搭建一个单机版的RocketMQ没有太大的实用价值，所以在这里**搭建一个双主 、 双从 、 无单点故障**的高可用 RocketMQ集群 。

#### 材料准备
两台物理机（这里我采用了两台虚拟机）
主机地址：
192.168.223.41 node01
192.168.223.42 node02

#### 配置步骤
**1.解压文件**
unzip rocketmq-all-4.3.2-bin-release.zip -d ./rocketmq-all-4.3.2-bin

**2.启动两机器的nameserver**
[root@node01 rocketmq-all-4.3.2-bin-release]# nohup sh bin/mqnamesrv &
[root@node02 rocketmq-all-4.3.2-bin-release]# nohup sh bin/mqnamesrv &

**3.broker配置文件**

**双master配置：**

[root@node01]#vim /rocketmq-all-4.3.2-bin-release/conf/2m-2s-sync/broker-a.properties

```java
namesrvAddr=192.168.223.41:9876;192.168.223.42:9876
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=SYNC_MASTER
flushDiskType=ASYNC_FLUSH
listenPort=10911
storePathRootDir=/home/rocketmq/store-a
```

[root@node02]#vim /rocketmq-all-4.3.2-bin-release/conf/2m-2s-sync/broker-b.properties
```java
namesrvAddr=192.168.223.41:9876;192.168.223.42:9876
brokerClusterName=DefaultCluster
brokerName=broker-b
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=SYNC_MASTER
flushDiskType=ASYNC_FLUSH
listenPort=10911
storePathRootDir=/home/rocketmq/store-b
```

**双slave配置：**
[root@node01]#vim /rocketmq-all-4.3.2-bin-release/conf/2m-2s-sync/broker-b-s.properties
```java
namesrvAddr=192.168.223.41:9876;192.168.223.42:9876
brokerClusterName=DefaultCluster
brokerName=broker-b
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SYNC_MASTER
flushDiskType=ASYNC_FLUSH
listenPort=10911
storePathRootDir=/home/rocketmq/store-b
```
[root@node02]#vim /rocketmq-all-4.3.2-bin-release/conf/2m-2s-sync/broker-a-s.properties
```java
namesrvAddr=192.168.223.41:9876;192.168.223.42:9876
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
listenPort=11011
storePathRootDir=/home/rocketmq/store-a
```
**4.启动4个broker：**
命令：nohup sh ./bin/mqbroker –c config_file &

[root@node01 rocketmq-all-4.3.2-bin-release]# nohup sh ./bin/mqbroker -c ./conf/2m-2s-sync/broker-a.properties &
[root@node01 rocketmq-all-4.3.2-bin-release]# nohup sh ./bin/mqbroker -c ./conf/2m-2s-sync/broker-b-s.properties &
[root@node02 rocketmq-all-4.3.2-bin-release]# nohup sh ./bin/mqbroker -c ./conf/2m-2s-sync/broker-b.properties &
[root@node02 rocketmq-all-4.3.2-bin-release]# nohup sh ./bin/mqbroker -c ./conf/2m-2s-sync/broker-a-s.properties &

jps查看效果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130155920435.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)
**5.查看集群状态**
可采用图形化界面管理
https://github.com/apache/rocketmq-externals/tree/master/rocketmq-console
图形化界面配置过程参考：
https://yq.aliyun.com/articles/486069?spm=5176.10695662.1996646101.searchclickresult.37e0a3dbrOPWp6


## 三、相关配置参数介绍

- namesrvAddr= 192.168.100.131:9876;  192.168.100.132:9876 
	NamerServer 的地址，可以是多个 。 
- brokerClusterName=DefaultCluster 
	Cluster 的地址，如果集群机器数比较多，可以分成多个 Cluster，每个 Cluster 供一个业	务 群使用 。 
-  brokerName=broker- a 
	Broker 的名称， Master 和 Slave 通过使用相同的 Broker 名称来表明相互关系，以说明某个 Slave 是哪个 Master 的 Slave。 
- brokerid=0
	一个 Master Barker可以有多个 Slave, 0表示 Master，大于 0表示不同 Slave 的 ID。 
- fileReservedTime=48 
	在磁盘上保存消息的时长，单位是小时，自动删除超时的消息 。 
- deleteWhen=04 
	与 fileReservedTim巳参数呼应，表明在几点做消息删除动作，默认值 04表 示凌晨 4点。 
- brokerRole=SYNC MASTER 
	brokerRole 有 3 种: SYNC MASTER、 ASYNC MASTER、 SLAVE。 
	前两个是master，第三个是slave。 Master的前缀代表主从复制方式，异步复制和同步	复制
- flushDiskType=ASYNC FLUSH 
	flushDiskType表示刷盘策略，分为SYNC_FLUSH和ASYNC_FLUSH两 种，分别代表同步刷 	盘和异步刷盘。 同步刷盘情况下，消息真正写人磁盘后再 返回成功状态;异步刷盘情况	下，消息写入 page_cache 后就返回成功状态 。 
- listenPort=l0911
	Broker监听的端口 号，如果一台机器上启动了多个 Broker， 则要设置不同 的端口号， 	避免冲突 。
- storePathRootDir=/home/rocketmq/store - a 
	存储消息以及一些配置信息的根目录 。 

## 四、配置过程踩过的一些坑
**坑1：**
下载不同版本rocketmq时，要注意其要求的jdk版本
原因，高版本rocketmq中有MaxMetaspaceSize参数，MaxMetaspaceSize为Java8中新引入的参数

**坑2**：
启动mqnamesev时候报：	
Java HotSpot 64-Bit Server VM warning: INFO: os::commit_memory(0x00000006c0000000, 2147483648, 0) failed; error='Cannot allocate memory' (errno=12)
则需要调整rocketMQ的内存

```java
vim bin/runserver.sh 
JAVA_OPT="${JAVA_OPT} -server -Xms128m -Xmx256m -Xmn256m -XX:PermSize=128m -XX:MaxPermSize=320m" 
```
-Xms 的值一定要比 -Xmx 要小不让，也会报错：
Initial heap size set to a larger value than the maximum heap size

启动mqbroker时报，则修改:
```java
vim bin/runbroker.sh
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m"
```


