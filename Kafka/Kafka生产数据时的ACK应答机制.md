# Kafka生产数据时的ACK应答机制 #

### 一、ACK应答机制

Kafak为用户提供了三种应答级别：all, leader, 0，分别对应1, 0, -1。

```Kafka
一句话总结：
-1 ：意味着producer需要得到follower确认后才发送下一条数据。持久性、可靠性最好，时延大、性能最差；
 1 ：意味着producer要等待leader成功收到数据并得到确认，才发送下一条message。此选项提供了较好的持久性较低的延迟性。
Partition的Leader死亡，follwer尚未复制，数据就会丢失；
 0 ：意味着producer不等待broker同步完成的确认，继续发送下一条(批)信息。最弱的可靠性，但是提供了最低的延迟。当服务器发生故障时，就很可能发生数据丢失。例如leader已经死亡，producer不知情，还会继续发送消息，broker接收不到数据就会发生数据丢失。
```

#### 1. acks: 0

这种应答机制提供了最低的时延，partition的leader在接收到消息，尚未写入磁盘就已经返回ACK，此时当leader发生故障的时候就会丢失数据。生产者发送完消息之后不会等待broker的任何确认消息，这种方式效率很高但可靠性大大降低。

#### 2. acks: 1（leader）

partition的leader落盘成功后返回ack，如果在follower同步成功之前leader故障，尽管 leader 已经落盘成功，但是 follower 的同步进度肯定是低于leader，这时故障，那么将会丢失 follower 还未同步 leader 那部分数据；这种模式下，具有一定的可靠性和效率，但是依旧有丢失数据的可能性。

![image-20240227112849952](http://image.huawei.com/tiny-lts/v1/images/96aea66d460374f503b132725f613dae_803x382.png)

#### 3. acks: -1（all）

partition的leader和follower全部落盘成功后才返回ack。但是如果在follower同步完成后，broker发送ack之前，leader发生故障，即选举出新的 leader，新的 leader 将再次落盘一次，那么会造成数据重复；这种模式下，效率是最低的，但是数据可靠性则最高。

![image-20240227112908214](http://image.huawei.com/tiny-lts/v1/images/2677f64f4d891a67f3e04afdd395b866_817x375.png)

#### 4. java api中对应参数

```java
// 设置acks
properties.put(ProducerConfig.ACKS_CONFIG, "all");

// 重试次数retries，默认是int最大值，2147483647
properties.put(ProducerConfig.RETRIES_CONFIG, 3);
```

### 二、强化记忆

```Kafka
ACK = 0：只发一次数据，不管leader是否接受成功
ACK = 1：只需要等待leader接受完成并确认即可发送下一条数据
ACK = -1：需要等待leader+follower所有副本的应答确认才发送数据
```
