## nsq的设计特点：
* 分布式的队列传输节点，能够避免传输节点的单点故障，
* 确保消息一定会被传输一次， 但是这样可能引起消息被多次传输
* bound the memory footprint of a single process (by persisting some messages to disk)
* 简化消费者和生产者的配置
* 提供了一个简单明确的升级路线
* 提升了效率

## nsqde的基本概念
* nsqd, 转发消息的核心, 负责接受消息， 将消息排队， 发送给客户端, 不同的nsqd之间的消息不会共享
* nsqlookupd ，只是提供client查询当前所有的nsqd的信息， 不会对实际的消息转发做任何的处理， nsqlookupd于nsqd之间在底层建立了tcp的长连接连接， 而默认监听的4160端口就是干这个用的，通过周期性的报告nsqd的状态，
* client 消费者， 需要配置nsqlookupd的地址
* nsqadmin 供管理员集中查看所有的集群，有一个webui
* topic， 属于nsqd， 就是一个消息流，目测是对业务的标志， 或者是为了将同一个物理管道进行逻辑细分的方式， 算是为了提高效率和细分资源，topic并不需要进行实现的配置，一个nsqd可能拥有多个topic
* channel， 属于nsqd，是发送给一个topic的逻辑组， 一个topic可能用拥有多个channel， 每个channel都能收到这个topic下面的所有的消息， 一般一个channel都会对应一个消费者， 而一个channel可以对应多个消费者，channale会将消息随机发送给下面的一个client，所以任意一个client收到的消息都不是完全的，
* channel和topic都是单独缓存消息的

## 分布式方案

* nsqd随意起， nsqlookup使用备份的方式
* 一个client只会同时对一个nsqd建立连接， 所以一旦一个nsqd连接， 那么就不会对其他的topic建立连接
* 只有再一个nsqd坏掉的时候，才会重新选择nsqd
* 那么client选择nsqd的原则是什么？


## nsq保证消息能够正常至少传输一次的方式是：
* 1: client表明已经可以接受消息
* 2: nsqd将消息发送出去， 同时将这个消息进行本地存储
* 3: client如果回复FIN 表示成功接受， 如果回复REQ， 表明需要重发， 如果没有回复， 则认为超时了， 进行重发
* 所以， 当nsqd异常关闭的时候， 没有来得及保存到本地的消息可能会丢失, 解决办法是讲同样的消息发送到两个nsqd中

    这里如何保证消息不丢失的？

## nsdlookup 如何路由请求    {
"channels": [
"nsq_to_file",
"c"
],
"producers": [
{
"remote_address": "127.0.0.1:58148",
"hostname": "safedev01v.add.corp.qihoo.net",
"broadcast_address": "safedev01v.add.corp.qihoo.net",
"tcp_port": 4150,
"http_port": 4151,
"version": "0.3.6"
},
{
"remote_address": "10.16.59.85:39652",
"hostname": "safedev02v.add.corp.qihoo.net",
"broadcast_address": "safedev02v.add.corp.qihoo.net",
"tcp_port": 4150,
"http_port": 4151,
"version": "0.3.7"
}
]
}

## 雪崩如何处理    

## 如何实现消息快四推送



## 对消费端APP的要求
* 幂等性需求


## 将消息如队列的过程
* 得到一个消息的对象， 并分配内存
* 对topic加读锁
* 获取发布消息的读锁，
* 讲消息发送给带缓存的队列


## 细节
* nsqlookupd的高可用是通过同时运行多个实例， 多个实例之间保持互备实现的。
* 由于消息至少会被发送一次， 则意味着消息可能会被发送多次， 客户端需要能够确定收到消息所执行的操作是幂等的，即收到一次与收到多次的影响一致
* 可以设置内存的使用大小， 但是并不建议将内存设置太小， 毕竟持久化是为了保证unclean关闭nsqd时，消息不会丢失
* nsq-chan 的信息就是保存在go-chan中的，



## 每一个topic包含三个协程：
* router： 从go-chan中读取新发布的消息，并讲消息保存在一个队列（ram or rom）中，
* messagePump
* DiskQueue 讲内存中的消息写入到磁盘，
* 如果一个topic没有订阅者（客户端），则该topic的内容就不会被diskqueue写入到磁盘中， 而是由DummyBackendQueue直接将消息丢弃掉

    ![](https://f.cloud.github.com/assets/187441/1698990/682fc358-5f76-11e3-9b05-3d5baba67f13.png)

    每一个队列都包含了两个基于时间排序的优先级队列， 这个是为了防止出现错误和消息的timeout， 而这两个队列是有两个协程进行监控的

    **尽量少地减少内存的使用是为了减小gc消耗的时间，**
### nsqd中减小GC的优化方案是：
* 避免[]byte转换string
* 重用缓存或者对象
* 预先分配slice的内存， 并且知道每个item的大小
* 避免使用interface{} 和封装的类型，
    >like a struct for a “multiple value” go-chan).
* 避免使用defer




## 效率
* client可以是一个rdy状态， 然后由nsqd想client推送消息。
    ![](http://media.tumblr.com/tumblr_mataigNDn61qj3yp2.png)
* clientde rdy状态会给nsqd的数字表示client期望收到的消息的数目，这样做的意义是啥？
* nsqd会周期性地向client发送心跳信息， 而心跳信息和rdy可以防止出现 head-of-line blocking
* 所有的网络IO操作都根据心跳的间隔设置了超时时间， 一旦超时， 正在发送的消息就会重新进入队列， 同时这个错误回被记录下来
* nsq 使用waitgroup 实现等待协程执行完成



## 问题
* 消息传输节点是通过lookup节点实现的高可用性， 那么如何避免lookup节点的单点故障
* 每个channel都会随机讲消息下发至其客户端，所以如果一个channel下面连接了多个client的时候， 每个client收到的消息都不是完全的，这样有什么好处呢？
* nsqlookup只是提供对nsqd查询，所以nsqlookup不会成为单点故障， 但是如何保障多个nsqd之间的负载是均衡的？
* 一个客户端要于所有的nsqd保持连接， 那么这样是否会存在性能底下的问题？
* 什么时候会出现消息发送多次的情况？
