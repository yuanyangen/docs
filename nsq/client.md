# nsq客户端的工作原理
  客户端分为两个部分， 一个是向nsqd中推送消息， 另外一个是从nsqd中消费消息
## 1： 客户端推送消息
  向使用http的方式，向nsqd所在的4151端口推送数据即可
## 2： 客户端从nsqd获取消息

* 一个客户端通过于nsqd建立一个tcp连接试下你订阅一个消息的topic中的channel， 客户端在于nsqd之间的tcp连接可以订阅一个topic 或者多个topic

### 建立连接
* 对客户端而言，使用nsqlookupd是可选的， 这说明客户端既要支持直接连接到nsqd，也要支持轮询nsqlookupd的实例，同时该轮询间隔需要能够可以配置
* 如果使用nsqlookupd， 客户端需要能够轮询所有的nsqlookupd实例， 通过访问nsqlookupd的lookup接口实现， 返回的json里面就有所期望的topic所在的nsqd的位置， 一旦发现了新机器， 就会于新发现的nsqd建立连接，
* client开始工作的时候， 会与每一个nsqd建立独立的tcp一个连接，并通过该tcp连接发送以下的消息：自己的ID，IDENTIFY命令， sub命令，count为1的RDY状态
* 客户端在以下的时候将会进行重连：如果客户端配置的是特定的nsqd， 那么在进行重发请求的时候， 就进行重连， 如果客户端是从nsqlookupd获取的nsqd， 那么只有在从nsqlookupd获取到这个nsqd的实例信息之后才进行重连
* IDENTIFY命令用于协商nsqd和client两边的配置， identify命令的内容包括：id，hostname， 心跳间隔等， 而nsqd回应的内容包括max-rd-count， 版本


### 数据流
* nsq中的数据流是异步的， 所以client也需要对数据进行异步的处理， 尼玛这个是啥意思？
* client 需要响应由nsqd定期发送的心跳信息，回应的方式就是任意的命令即可， 最简单的就是nop

### client对消息的处理：
* FIN 客户端成功收到这个消息， nsqd可以直接丢弃掉这个消息的缓存了，
* REQ 客户端接受消息失败， 需要nsqd重发
* TOUCH 客户端如果需要更多的时间处理这个消息， 就发送touch命令。
* 如果客户端没有发送消息， 则nsqd会自动对消息进行重传




### client的RDY管理
* 维护max_in_flight， 这个是实现配置好的，
* 维护RDY count, 这个是针对每一个连接来的。
* nsqd维护了一个max_rdy_count


* 客户端在启动的时候会给nsqd发送一个rdy的消息， 其中指明了客户端能够接受的消息的数目， 当rdy的数目达到max-in-flight的时候，客户端重新发送rdy update
* 客户端应该尽量保证max-in-flight均匀分布在所有的连接上，
* 客户端维护的rdy不应该超过max_in_flight
* 对于nsqd而言， 如果client发送的rdy的数字大于自身的max_rdy_count， 则该连接会被关闭



* 客户端直接配置的max_in_flight影响了整体的nsq的性能， 这个类似于连续的arq于普通的arq之间的区别

### client的消息失败管理

* client 需要控制传输失败的消息， 需要进行重传以及重传次数的限制， 而在nsq中， 这些都是由客户端决定的， 就是说重传控制都是客户端在协议中决定的， 这个协议似乎有点意思
* client通过发送rdy0 给nsqd实现暂时的不发送数据

![](http://media.tumblr.com/7adbf06362cc6530153ef35b4dacf2cb/tumblr_inline_mmjev3stkE1qz4rgp.png)

### nsqd的消息重新排队机制

* nsqd 在收到消息事失败的通知后， 如何讲失败的消息进行重新排队，


# 问题
* 这个Message Flow Starvatio是干嘛用的？
