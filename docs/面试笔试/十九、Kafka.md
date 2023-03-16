# 十九、Kafka

## <strong>📌 Kafka 为什么这么快</strong>

1. 零拷贝
2. 顺序写
3. 操作系统层面的 page cache 技术
4. 稀疏索引
5. 批量读，批量写

参考：

- [动画讲解：Kafka 为什么快之零拷贝技术 -- mmap #Kafka - 抖音](https://www.douyin.com/video/7198851465329331514)

## <strong>📌 Kafka 什么情况下会 rebalance</strong>

### 什么是 Rebalance

Rebalance 是 Kafak Consumer 就如何分配 Partition 消费，达成共识的一个过程

### 触发 Rebalance 的条件是什么

1. 同一个 Group 有新的 Consumer 加入
2. Consumer 下线：包括主动取消订阅，超时下线
3. Coordinator 挂了，集群需要选出新的 Coordinator
4. Topic 变动：包括 Topic 变动和 Partition 数量变动

### Rebalance 危害是什么

Rebalance 期间，集群所有节点都会停止工作，参与 Rebalance

## <strong>📌 能简单说一下 rebalance 过程吗？</strong>

主要的流程如下：

1. 发送 GCR 请求寻找 Coordinator：这个过程主要会向集群中负载最小的 broker 发起请求，等待成功返回后，那么该 Broker 将作为 Coordinator，尝试连接该 Coordinator
2. 发送 JGR 请求加入该组：当成功找到 Coordinator 后，那么就要发起加入 group 的请求，表示该 consumer 是该组的成员，Coordinator 会接收到该请求，会给集群分配一个 Leader（通常是第一个），让其负责 partition 的分配
3. 发送 SGR 请求：JGR 请求成功后，如果发现当前 Consumer 是 leader，那么会进行 partition 的分配，并发起 SGR 请求将分配结果发送给 Coordinator;如果不是 leader，那么也会发起 SGR 请求，不过分配结果为空

## <strong>📌 Rebalance 有什么影响</strong>

Rebalance 本身是 Kafka 集群的一个保护设定，用于剔除掉无法消费或者过慢的消费者，然后由于我们的数据量较大，同时后续消费后的数据写入需要走网络 IO，很有可能存在依赖的第三方服务存在慢的情况而导致我们超时。Rebalance 对我们数据的影响主要有以下几点：

1. 数据重复消费: 消费过的数据由于提交 offset 任务也会失败，在 partition 被分配给其他消费者的时候，会造成重复消费，数据重复且增加集群压力
2. Rebalance 扩散到整个 ConsumerGroup 的所有消费者，因为一个消费者的退出，导致整个 Group 进行了 Rebalance，并在一个比较慢的时间内达到稳定状态，影响面较大
3. 频繁的 Rebalance 反而降低了消息的消费速度，大部分时间都在重复消费和 Rebalance
4. 数据不能及时消费，会累积 lag，在 Kafka 的 TTL 之后会丢弃数据 上面的影响对于我们系统来说，都是致命的。

## <strong>📌 怎么解决 rebalance 中遇到的问题呢？</strong>

要避免 Rebalance，还是要从 Rebalance 发生的时机入手。我们在前面说过，Rebalance 主要发生的时机有三个：

- 组成员数量发生变化
- 订阅主题数量发生变化
- 订阅主题的分区数发生变化

后两个我们大可以人为的避免，发生 rebalance 最常见的原因是消费组成员的变化。

- 有两个重要参数的设置，一个是超时时间：`session.timeout.ms` 另外一个是心跳时间：`heartbeat.interval.ms`；一个请求的处理时间如果超过了超时时间，那么就会把它踢出 Group，会发生 Rebalance；所以心跳时间的设置必须小于超时时间的设置

## <strong>📌 如何保证 kafka 顺序消费</strong>

这个在我看来是一个伪命题，如果要保证顺序消费为啥要用 kafka 呢，只是需要做到异步或者解耦？如果一定要做到顺序消费，肯定是可以的，但是这个浪费资源，因为 kafka 就是针对高并发大吞吐量而生，下面说一下顺序消费方案：1、一个 topic、一个 partition、一个线程 2、一个 topic、n 个 partition、n 个线程，这里生产时需要根据需求将需要排序的数据发送到指定的 message key

## <strong>📌 说一下 Kafka 消息积压如何处理</strong>

Kakfa 消息积压常见于 Consumer 端消息积压，常见的处理方式有两种

1. 增加 partition 数量，同时增加 consumer 数量，提高并行度
2. Consumer 端单条拉取变为批量拉取（max.poll.records），并且调整单次拉取的数据量（fetch.max.bytes），提高消费吞吐量

##### 常用参数

| 参数             | 含义                                                                                                                                                                                                                                                             |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| fetch.max.bytes  | 默认 Default:52428800（50m）。消费者获取服务器端一批消息最大的字节数。如果服务器端一批次的数据大于该值（50m）仍然可以拉取回来这批数据，因此，这不是一个绝对最大值。一批次的大小受 message.max.bytes （broker config）or max.message.bytes （topic config）影响。 |
| max.poll.records | 一次 poll 拉取数据返回消息的最大条数，默认是 500 条                                                                                                                                                                                                              |

## 📌 说一下 Kafka Ack 机制

Kafka Ack  机制是 broker 收到 producer 发送的消息后，对 producer 进行应答的机制，主要作用是确保 producer 发送的消息 brocker 能够正常收到；Ack 的值有 1、0、-1

- ack=0 默认：broker 的 leader 节点收到消息后立即回复，不会落盘；优点：吞吐最高，缺点：会丢消息，比如 leader 节点挂掉
- ack=1：broker 的 leader 节点收到消息后，先落盘，再回复；优点：兼顾效率和消息丢失的问题；缺点：消息丢失仍然存在，比如：收到消息后，leader 节点挂掉
- ack=-1：broker 的 leader 节点收到消息后，ISR 列表中所有节点都落盘成功后，再回复；优点：消息不会丢失；缺点：效率较低，需要等待 ISR 列表中所有节点都落盘才能回复

## 📌 说一下 kafka 通信过程原理

参考：

- [面试必备：kafka 夺命连环 11 问](https://www.bmabk.com/index.php/post/81833.html)

## 📌 说一下 Consumer 分区策略

- Range（默认）：把所有 Partition 平均分配给所有 Consumer，多余的分给第一个 Consumer
- RoundRobin：计算 hashCode 进行分配
- Sticky：粘性分区
- CooperativeSticky

参考：

- [十九、Consumer 分区分配及再平衡](https://jqz3pp5nv2.feishu.cn/wiki/wikcnSWyyKYJPOu461NSxmNsNHc)

## 📌 说一下 Kafka 消息如何不丢失

#### 消息生产者

- ACK 机制：ACK = ALL

#### Broker

- 分区副本大于等于 2（--replication-factor 2）
- ISR 里应答的最小副本数量大于等于 2 （min.insync.replicas= 2）

#### Consumer

- 关闭自动提交，开启手动提交

## 📌 说一下 Kafka 消息如何不重复不丢失（精确一次）

### producer

- acks 设置为-1 （acks=-1）
- 幂等性（enable.idempotence = true）+ 事务

### broker

- 分区副本大于等于 2（--replication-factor 2）
- ISR 里应答的最小副本数量大于等于 2 （min.insync.replicas= 2）

### Consumer

- 事务 + 手动提交 offset（enable.auto.commit=false）
- 消费者输出的目的地必须支持事务（MySQL、Kafka）

参考：

- [二十五、Kafka 常见问题](https://jqz3pp5nv2.feishu.cn/wiki/wikcnuJw0KuxSicJRa0PuRfhQuf)

## 📌 说一下 Kafka 如何实现有序消息

- Broker 端保证单分区即可，Producer 端保证顺序发送，即可保证 Consumer 端顺序消费

## 📌 在有序消费的同时，如何提高吞吐量？满足高并发要求

## 参考资料

- [9 个 Kafka 面试题，你会几个?-技术圈](https://jishuin.proginn.com/p/763bfbd64a48)
- [Kafka 面试题系列之进阶篇 - 墨天轮](https://www.modb.pro/db/149986)
