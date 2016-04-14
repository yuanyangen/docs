## nsq 的安装
到官网下载最新的自己平台的二进制包`http://nsq.io/deployment/installing.html` 解压出来即可使用

## 从源码编译安装nsq
* 创建nsq的源码目录， 确保这个文件中有你想安装的对应版本的Godeps文件   
        
        https://github.com/nsqio/nsq/blob/v0.3.7/Godeps
   
* 下载依赖：
        
        gpm install

* 下载nsq源码
        
        go get github.com/nsqio/nsq/...

* 在`src/github.com/nsqio/nsq/`下面有nsq的所有源码,
        
        make 会在当前路径的build下面创建所有的二进制文件
        也可以进入app下面，对所有的文件进行自定义的编译

        
## 不使用lookup的时候：
* 启动nsqd：

        ./nsqd

* 向nsqd中发送消息， 另开一个shell， 在这个机器中执行

        curl -d 'hello world 1' 'http://127.0.0.1:4151/put?topic=test'

* 消费者消费消息， 在这台机器上面执行以下的php脚本

## 使用nsqlookup
* 启动nsqlookip

        ./nsqlookupd

* 启动nsqd

        nsqd --lookupd-tcp-address=127.0.0.1:4160

* 管理消息

        nsqadmin --lookupd-http-address=127.0.0.1:4161

* 产生消息

        curl -d 'hello world 1' 'http://127.0.0.1:4151/put?topic=test'

* 消费消息       

        nsq_to_file --topic=test --output-dir=/tmp --lookupd-http-address=127.0.0.1:4161


## 使用golang客户端

### 准备环境

    go get github.com/mreiferson/go-snappystream
    go get github.com/nsqio/go-nsq

### 生产者

    package main

    import (
	    "fmt"
    	"strconv"

	    "github.com/nsqio/go-nsq"
    )

    func main() {
	    producer, err := nsq.NewProducer("127.0.0.1:4150", nsq.NewConfig())
	    defer producer.Stop()
	    if err != nil {
		    fmt.Println(err.Error())
    	}
	    for i := 0; i < 10; i++ {
		    err = producer.Publish("test", []byte("testing"+strconv.Itoa(i)))
    		if err != nil {
	    		fmt.Println(err.Error())
		    }
    	}
    }



### 消费者


    package main

    import (
      "fmt"

      "github.com/nsqio/go-nsq"
    )

    func main() {
      consumre, err := nsq.NewConsumer("test", "c", nsq.NewConfig())
      if err != nil {
        fmt.Println(err.Error())
      }

      nsqHand := &hand{}
      nsqHand.msgChan = make(chan *nsq.Message, 100)
      consumre.AddHandler(nsq.HandlerFunc(nsqHand.handler))
      err = consumre.ConnectToNSQLookupd("127.0.0.1:4161")
      if err != nil {
        fmt.Println(err.Error())
      }

      for {
        select {
        case msg := <-nsqHand.msgChan:
          fmt.Println(string(msg.Body))
        }
      }
    }

    type hand struct {
      msgChan chan *nsq.Message
    }

    func (this *hand) handler(message *nsq.Message) error {
      this.msgChan <- message
      return nil
    }

## 其他工具的使用

    nsq

