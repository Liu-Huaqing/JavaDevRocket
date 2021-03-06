# 分布式消息队列
## 什么是消息队列？

## 为什么使用消息队列啊？消息队列的特性是什么？

消息队列的特性：`业务无关`，`FIFO`，`容灾`，`性能`

当系统中出现`“生产“和“消费“的速度或稳定性等因素不一致`的时候，就需要消息队列，作为抽象层，弥合双方的差异。“ 消息 ”是在两台计算机间传送的数据单位。消息可以非常简单，例如只包含文本字符串；也可以更复杂，可能包含嵌入对象。消息被发送到队列中，“ 消息队列 ”是在消息的传输过程中保存消息的容器 。

举几个例子

* `业务系统触发短信发送申请`，但短信发送模块速度跟不上，需要将来不及处理的消息暂存一下，缓冲压力。
* `调用远程系统`下订单成本较高，且因为网络等因素，不稳定，`攒一批一起发送`。
* `任务处理类的系统`，先把用户发起的任务请求接收过来存到消息队列中，然后后端开启多个应用程序从队列中取任务进行处理。

## 消息队列有什么优点和缺点啊？

* `异步化、提高系统响应速度`

使用了消息队列，`生产者一方，把消息往队列里一扔，就可以立马返回，响应用户了`。无需等待处理结果。

处理结果可以让用户稍后自己来取，如医院取化验单。也可以让生产者订阅（如：留下手机号码或让生产者实现listener接口、加入监听队列），有结果了通知。获得约定将结果放在某处，无需通知。

* `解耦、提高系统稳定性`

`考虑电商系统下订单，发送数据给生产系统的情况`。电商系统和生产系统之间的网络有可能掉线，生产系统可能会因维护等原因暂停服务。如果不使用消息队列，电商系统数据发布出去，顾客无法下单，影响业务开展。两个系统间不应该如此紧密耦合。应该通过消息队列解耦。同时让系统更健壮、稳定。

* `消除峰值`

## 如何保证消息队列的高可用啊？

`以 kafka 为例`，来解释高可用的保证策略，Kafka的高可用性的保障来源于其`健壮的副本（replication）策略`，N个副本中，有一个Leader，余下N-1个Follower。Kafka的写操作只在Leader副本上进行，然后同步给 Follower。当 Leader 当机，通过合理的 FailOver 机制 Follower 会重新来承担 Leader 的角色，继续支撑对外服务。

核心的两个 HA 功能：
* `Data Replication`
* `Broker FailOver`

=====================================详细解释====================================

`Replication 策略的数据存储格式`

* 一个`Topic`可以分成多个`Partition`，而一个`Partition`物理上由多个`Segment`组成。
* `Segment分2部分：索引文件和数据文件`。索引文件保存元数据，记录了消息在数据文件中的偏移（offset），消息有固定物理结构，保证了正确的读取长度。
* Segment文件带来好处：方便过期文件清理。`只需要整体删除过期的Segment`。以追加的方式写消息，顺序写磁盘极大提高了效率。
* `读取某 offset 消息`的步骤变为：通过二分查找，找到 offset 所在 Segment。通过 Segment 的索引文件，找到 offset 所在数据文件的物理偏移。读取数据。

`Replication 复制与同步`

从外部来看Partition类似一个不断增长，存储消息的数组，每个Partition有一个类似MySQL binlog的文件用来记录数据的写入。
有两个新名词，`HW（HighWatermark）`表示当前 Consumer 可以看到 Partition 的 offset 位置，`LEO（LogEndOffset）`表示当前 Partition 最新消息的offset，各个副本单独维护。为了提高消息可靠性，Partition 有N个副本。

`N个副本中，有一个Leader，余下N-1个Follower。Kafka的写操作只在Leader副本上进行`

通常这种副本写有两种方式：

* `Leader写日志文件成功即返回成功`。这样如果Follower在同步完数据前Leader当机，数据丢失。这种方式带来较高效率。
* `Leader等待Follwer写日志成功并收到返回的acks后，才返回成功`。这样Leader当机，重新选举的Leader与当机Leader数据一致，数据不丢失。但因为要等待Follwer返回，效率较慢。一般采用少数服从多数的选举方式，如果要应对f个副本当机，则至少需要2f+1个副本并使中的f+1个写成功。
* `ISR（In-Sync Replication）机制`（Kafka 使用），Leader维护一个副本队列（包含Leader自己），会及时将慢响应的Follwer剔除，并将追上Leader数据的Follower重新加入副本队列。这样要保证数据高可靠所需要的副本数更少。比如应对2台机器的当机，ISR机制只需要3个副本。而上述机制2则需要2*2+1个副本。这样有效节约了大约一半的存储空间。Leader当机，新的Leader是从ISR中按顺序选出。Leader恢复后成为Follower，删除上一个HW后所有数据后，从新的Leader进行同步。

## Kafka Broker Fail Over 过程
* TODO


## 如何保证消息不被重复消费啊？

Kafka 使用 Offset 机制来进行消息消费的跟踪。在 Consumer 获取数据的时候，更新相应的 Offset。具体 Offset 由谁来保存，保存在哪儿，都可以灵活配置。
在 Kafka 历史变迁的过程中，主要有以下几类的 Consumer:
* `HighLevel-Consumer`: 比较自动化的版本，但`不支持自己保存 Offset`,有`惊群效应和脑裂问题`。
* `Simple-Consumer`: 非常简单的版本，自己需要做很多事情。
* `KafkaConsumer-subscribe`: 在拥有 `HighLevel-Consumer` 的特性的基础上，支持了`自己保存 Offset`，`接近完美`。

通过 Offset 的合理控制，可以有效避免消息的重复消费:
消费消息和修改 Offset 的先后

1. `先修改 Offset，再消费消息`
这种情况下，一旦 Offset 被修改，就已经脱离了 Kafka 的范围，由`客户端来唯一保证消息被正确的唯一的消费和处理`。
由 Kafka 保存 Offset 就是这一类情况

2. `先消费消息，再修改 Offset`
这种情况下，客户端定义来 Kafka 拿一批数据消费，消费完成后，再修改 Offset。客户端假定`消息可以被重复消费`，或者`客户端有自己的去重机制`。

相关配图：
![Kafka Consumer版本变迁](./consumer-history.jpeg)

## kafka, activemq, rabbitmq, rocketmq, redis 都有什么优点，缺点啊？或者是然后选择消息中间件？

整体思路
* 是否支持 `JMS API规范`
* 发送的数据是  `Data or Cmd`，一般 Data 处理起来会较快（外部依赖少），Cmd 处理起来较慢（外部依赖多）
* Consumer 可以分为 `快和慢`，`稳定和不稳定`

Kafka 的特点是：
* message 被放到 topic 的 partition 中；
* partition 中的 消息可以看做是一个很长的 stream；
* consumer 负责遍历这个 stream;
* partition 中的消息遍历是有序的;

Kafka 的优势：
* 消费速度非常快速，达到单台机器 10万/每秒

Kafka 的劣势：
* `每个 partition 只能有一个 consumer`（in the consumer group）；
* 适合与快速 consumer 合作，`不适合慢速 consumer，容易让 partition 中的消息堆积`
* 由于底层实现机制的原因，`实现一个 partition 支持多 consumer 不容易`
* `没有 AMQP 的特点`

综上，kafka 适合于`高吞吐量、弱事务性、非事务性的消息传递`



RabbitMQ 是很传统的 Message Queue 的工作方式：
* 底层`实现 AMQP`;
* Producer 发送消息给 Queue;
* 多个 Consumers 同时连接一个 Queue；
* `消息会被分布到多个 Consumer 上来消费`；
* 如果只有一个 Consumer，消息投递顺序是可以保证的；

RabbitMQ 优势：
* `遵循 AMQP 协议`，具备各种 AMQP 的优点（需深入）；
* 借助erlang的特性在`可靠性、稳定性和实时性上比别的 MQ 做得更好`

`RabbitMQ 由于实现了 AMQP，适用于商业级别的消息通信`。

================================= 其他 Message Queue =================================
RocketMQ, ActiveMQ 不熟悉。

AMQP 协议特点（理解很浅显）：
* 容易理解和使用；
* 安全；
* 高保真 - 明确的`消息投递语意`覆盖（At Least Once, At Most Once, Exactly Once），明确的`消息顺序语意`, 明确的`失败语意`；
* 互通性；

Redis 只是一个很简单的内存级别的 pub-sub，不支持消息持久化;

## 如果让你写一个消息队列，该如何进行架构设计啊？说一下你的思路.
* TODO
* 消息队列/中间件是一个很核心的企业组件，值得在多深入研究之后再来重写本章


引用
* [@运维那点事](http://www.ywnds.com/?p=5791)
* [@To be, or not to be](https://www.cnblogs.com/tuhooo/p/8109083.html)
* [@Kafka高可用原理](https://www.cnblogs.com/gm-201705/p/7795676.html)
* [@Kafka设计解析(1)-Kafka High Availability（上）](http://www.jasongj.com/2015/04/24/KafkaColumn2/)
* [@Kafka设计解析(2)-Kafka High Availability（下）](http://www.jasongj.com/2015/04/24/KafkaColumn2/)
* [@Kafka设计解析(4)-KafkaConsumer设计解析](http://www.jasongj.com/2015/08/09/KafkaColumn4/)
* [@Kafka Consumer各版本分析总结](http://blog.csdn.net/cymvp/article/details/69554569)
* [@所有的 message queues](http://queues.io/)
* [@What is AMQP](https://www.amqp.org/about/what)
* [@AMQP 架构](http://www.amqp.org/product/architecture)
* [@kafka or rabbitmq](https://yurisubach.com/2016/05/19/kafka-or-rabbitmq/)

