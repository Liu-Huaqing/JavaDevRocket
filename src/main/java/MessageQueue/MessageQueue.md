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

## 如何保证消息队列的高可用啊？如何保证消息不被重复消费啊？

## kafka, activemq, rabbitmq, rocketmq, redis 都有什么优点，缺点啊？

## 如果让你写一个消息队列，该如何进行架构设计啊？说一下你的思路.


引用
* [@运维那点事](http://www.ywnds.com/?p=5791)
* [@To be, or not to be](https://www.cnblogs.com/tuhooo/p/8109083.html)
