---
title: 消息中间件RabbitMq
date: 2020-09-14 15:08:04
tags: 消息队列
excerpt: 转载 18、消息中间件RabbitMq
---

## [3_五种队列模式.xmind](https://uploader.shimo.im/f/dGqVYSa3LHYKfmyv.xmind)[4_消息确认机制.xmind](https://uploader.shimo.im/f/xiZWFxbBG6sUBCtt.xmind)[5_消息应答与持久化.xmind](https://uploader.shimo.im/f/OvsJwp2Y3SsuHfmH.xmind)[课程18预习_消息中间件RabbitMq.xmind](https://uploader.shimo.im/f/EJhLHdcFRBIWX3CA.xmind)[1_RabbitMq简介.xmind](https://uploader.shimo.im/f/JEdIJLdockkOQ0OR.xmind)[2_RabbitMq安装与组件.xmind](https://uploader.shimo.im/f/NC8GfDrAaKkPMP9i.xmind)

**消息与消息队列**

消息（Message）是指在应用间传送的数据。消息可以非常简单，比如只包含文本字符串，也可以更复杂，可能包含嵌入对象。

消息队列（Message Queue）是一种应用间的通信方式，消息发送后可以立即返回，由消息系统来确保消息的可靠传递。消息发布者只管把消息发布到 MQ 中而不用管谁来取，消息使用者只管从 MQ 中取消息而不管是谁发布的。这样发布者和使用者都不用知道对方的存在。

## 使用消息队列的情景例子

以常见的订单系统为例，用户点击【下单】按钮之后的业务逻辑可能包括：扣减库存、生成相应单据、发红包、发短信通知。在业务发展初期这些逻辑可能放在一起同步执行，随着业务的发展订单量增长，需要提升系统服务的性能，这时可以将一些不需要立即生效的操作拆分出来异步执行，比如发放红包、发短信通知等。这种场景下就可以用 MQ ，在下单的主流程（比如扣减库存、生成相应单据）完成之后发送一条消息到 MQ 让主流程快速完结，而由另外的单独线程拉取MQ的消息（或者由 MQ 推送消息），当发现 MQ 中有发红包或发短信之类的消息时，执行相应的业务逻辑。

## 应用场景

异步处理

应用解耦

流量削锋

日志处理

消息通讯

## RabbitMq教程

### AMQP

即Advanced Message Queuing Protocol,一个提供统一消息服务的应用层标准高级消息队列协议,是应用层协议的一个开放标准,为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。Erlang中的实现有 RabbitMQ等。

### RabbitMq维基百科

MQ全称为Message Queue, 消息队列（MQ）是一种应用程序对应用程序的通信方法。应用程序通过读写出入队列的消息（针对应用程序的数据）来通信，而无需专用连接来链接它们。消息传递指的是程序之间通过在消息中发送数据进行通信，而不是通过直接调用彼此来通信，直接调用通常是用于诸如远程过程调用的技术。排队指的是应用程序通过 队列来通信。队列的使用除去了接收和发送应用程序同时执行的要求。其中较为成熟的MQ产品有IBM WEBSPHERE MQ等等。


* [https://www.rabbitmq.com/getstarted.html](https://www.rabbitmq.com/getstarted.html)

![图片](https://images-cdn.shimo.im/qgubi7bYNV8EQbnx/image.png!thumbnail)

## rabbitMq安装

第一步、Erlang语言环境安装


* [http://www.erlang.org/downloads](http://www.erlang.org/downloads)

![图片](https://images-cdn.shimo.im/nLmjjYHoHgoGYQ3P/image.png!thumbnail)



安装erlang环境，看最新的RabbitMq支持的版本支持erlang的版本在20.3~22.0.*之间，为了通用后面的版本，我选了erlang的21.3.*版本安装

![图片](https://uploader.shimo.im/f/hh9u2y4wvrklK5YS.png!thumbnail)

于是有了下面步骤，安装官方的提示打开[https://github.com/rabbitmq/erlang-rpm](https://github.com/rabbitmq/erlang-rpm)

1、安装erlang环境

首先在打开/etc/yum.repos.d/rabbitmq-erlang.repo，注意手动创建rabbitmq-erlang.repo。

在rabbitmq-erlang.repo输入以下内容，保存，然后执行yum install erlang命令。

```
[rabbitmq-erlang]
name=rabbitmq-erlang
baseurl=https://dl.bintray.com/rabbitmq-erlang/rpm/erlang/21/el/7
gpgcheck=1
gpgkey=https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
repo_gpgcheck=0
enabled=1
```
然后输入erl，提示已经可以了。
2、安装RabbitMq，首先需要看版本，

打开[https://www.rabbitmq.com/install-rpm.html#downloads](https://www.rabbitmq.com/install-rpm.html#downloads)

![图片](https://uploader.shimo.im/f/qsM9lgQH6Pg51NYy.png!thumbnail)

```
#下载rabbitmq的rpm
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.15/rabbitmq-server-3.7.15-1.el7.noarch.rpm
yum install rabbitmq-server-3.7.15-1.el7.noarch.rpm
#完成后启动服务：
service rabbitmq-server start
#可以查看服务状态：
service rabbitmq-server status
```

3、、windows中安装RabbitMq


* [https://www.rabbitmq.com/install-windows.html](https://www.rabbitmq.com/install-windows.html)

![图片](https://images-cdn.shimo.im/VCBu8i6TwpYXXTg1/image.png!thumbnail)

4、启动RabbitMq

RabbitMq默认用户guest只能localhost访问，所以我们需要新建一个用户，命令如下：

```
新建：
rabbitmqctl add_user username password
比如：rabbitmqctl add_user root admin
授权：
rabbitmqctl set_user_tags username role
rabbitmqctl set_user_tags root administrator
启动界面管理插件
rabbitmq-plugins enable rabbitmq_management
可以在登录后在界面中管理用户和授权
```
5、启动RabbitMq（windows）


* 安装RabbitMQ 网页插件，在cmd控制面板中进入RabbitMQ 的安装目录下的rabbitmq_server-3.7.3\sbin文件夹，(D:\Program Files\RabbitMQServer\rabbitmq_server-3.7.3\sbin)，输入: rabbitmq-plugins.bat enable rabbitmq_management
* 默认会提供一个默认用户guest，密码也是guest，线上环境需要创建一个新用户，并把guest用户删除。
* 默认账号密码：guest  /  guest，直接打开localhost:15672可以登录

![图片](https://images-cdn.shimo.im/oh0uhfzhEwEfuG3V/image.png!thumbnail)

第四步、添加用户和授权


* 开始新建用户：重新通过cmd控制面板中进入RabbitMQ 的安装目录下的rabbitmq_server-3.7.3\sbin文件夹，(D:\Program Files\RabbitMQServer\rabbitmq_server-3.7.3\sbin)，开始新建用户，输入：rabbitmqctl.bat add_user username password，即可成功添加用户

![图片](https://images-cdn.shimo.im/eswMr2ZEx0cfSmzS/image.png!thumbnail)


* 授权管理员命令：rabbitmqctl.bat set_user_tags root administrator

![图片](https://images-cdn.shimo.im/ilyVG8FiW9wD0mQq/image.png!thumbnail)


* 授权命令： rabbitmqctl.bat set_permissions -p / root .* .* .*

![图片](https://images-cdn.shimo.im/XNq4Iz0K9RstAjK3/image.png!thumbnail)


* 停止/启动RabbitMQ服务（需要以管理员身份使用命令行）: net stop RabbitMQ && net start RabbitMQ

![图片](https://images-cdn.shimo.im/gyBH3B8sHjQtf1PN/image.png!thumbnail)


* RabbitMq界面。（[http://192.168.1.101:15672/](http://192.168.1.101:15672/#/users)）

![图片](https://images-cdn.shimo.im/GNv2X1NcEsECKB8V/image.png!thumbnail)

## RabbitMq命令大全

启动监控管理器：rabbitmq-plugins enable rabbitmq_management

关闭监控管理器：rabbitmq-plugins disable rabbitmq_management

启动rabbitmq：rabbitmq-service start

关闭rabbitmq：rabbitmq-service stop

查看所有的队列：rabbitmqctl list_queues

清除所有的队列：rabbitmqctl reset

关闭应用：rabbitmqctl stop_app

启动应用：rabbitmqctl start_app

用户和权限设置（后面用处）

添加用户：rabbitmqctl add_user username password

分配角色：rabbitmqctl set_user_tags username administrator

新增虚拟主机：rabbitmqctl add_vhost vhost_name

将新虚拟主机授权给新用户：rabbitmqctl set_permissions -p vhost_name username '.*' '.*' '.*'

## 概念理解

1.**Broker**：简单来说就是消息队列服务器实体。

2.**Exchange**：消息交换机，它指定消息按什么规则，路由到哪个队列。

3.**Queue**：消息队列载体，每个消息都会被投入到一个或多个队列。

4.**Binding**：绑定，它的作用就是把exchange和queue按照路由规则绑定起来。

5.**Routing Key**：路由关键字，exchange根据这个关键字进行消息投递。

6.**vhost**：虚拟主机，一个broker里可以开设多个vhost，用作不同用户的权限分离。

7.**producer**：消息生产者，就是投递消息的程序。

8.**consumer**：消息消费者，就是接受消息的程序。

9.**channel**：消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务。

## 五种队列模式


* P：消息的生产者
* C：消息的消费者
* 红色：队列

**应答机制ack**

**持久化机制durable**


* rabbitMq不允许一个已存在的队列重新定义持久化参数。
### 简单队列

简单队列的生产者和消费者关系一对一

项目代码：simpleQueue()；

![图片](https://images-cdn.shimo.im/bY6CrBOMsRUu7N8B/image.png!thumbnail)

### 工作队列

一个生产者、2个消费者。

但MQ中一个消息只能被一个消费者获取。即消息要么被C1获取，要么被C2获取。这种模式适用于类似集群，能者多劳。性能好的可以安排多消费，性能低的可以安排低消费。

默认：轮询分发

其他：公平分发


* 使用公平分发，必须关闭自动应答，改为手动应答。
* 消费者端添加代码：channel.basicQos(1);

项目代码：

![图片](https://images-cdn.shimo.im/ZhqgopRemeoJtPez/image.png!thumbnail)

### 发布订阅模式

这种模式可以满足消费者发布一个消息，多个消费者消费同一信息的需求。

广播给所有的接收者。

![图片](https://images-cdn.shimo.im/UXm6l2L9VVgjVElB/image.png!thumbnail)


1. 一个生产者，多个消费者
2. 每个消费者都有自己的队列
3. 生产者没有把消息发送到队列，而是发送到交换机exchange
4. 每个队列都绑定到交换机
5. 生产者发送的消息经过交换机，到达队列就能实现一个消息被多个消费者消费

* 伪代码--生产者
    * 创建mq链接、通道
    * 声明**交换机--****fanout****模式**，把消息发送到交换机上，（注意：交换机没有存储信息功能，不能持久化消息，因此，如果交换机没有绑定队列，那么消息就丢失了）
* 伪代码--消费者
    * 创建mq链接、通道
    * 声明队列
    * 把对列绑定到上交换机
    * 消费者监听该队列的通道
### 路由模式Routing

路由模式是在订阅模式基础上的完善，可以在生产消息的时候，加入Key值，与key值匹配的消费者消费信息。

![图片](https://images-cdn.shimo.im/HMJMnPkNZucvef58/image.png!thumbnail)


* 伪代码--生产者
    * 创建mq链接、通道
    * 声明**交换机--****direct****模式**，把消息发送到交换机上，并指定**routingKey**，（注意：交换机没有存储信息功能，不能持久化消息，因此，如果交换机没有绑定队列，那么消息就丢失了）
* 伪代码--消费者
    * 创建mq链接、通道
    * 声明队列
    * 把**对列绑定到上交换机，并指定routingKey**，多个就绑定多个routingKey
    * 消费者监听该队列的通道
### 通配符模式--Topic模式

通配符模式是在路由模式的升级，他允许key模糊匹配。*代表一个词，#代表一个或多个词。通过通配符模式我们就可以将C1对应的一个key准确定为item.add。而C2我们就不需要一一写出key值，而是用item.#代替即可。


* * 表示匹配一个
* # 表示匹配一个或多个

![图片](https://images-cdn.shimo.im/aGZv6ddx5LABsUYv/image.png!thumbnail)


* 伪代码--生产者
    * 创建mq链接、通道
    * 声明**交换机--****topic****模式**，把消息发送到交换机上，并指定**routingKey**（注意：交换机没有存储信息功能，不能持久化消息，因此，如果交换机没有绑定队列，那么消息就丢失了）
* 伪代码--消费者
    * 创建mq链接、通道
    * 声明队列
    * 把**对列绑定到上交换机，并指定routingKey（可以用*和#来进行模糊匹配）**
    * 消费者监听该队列的通道
### RPC

（略）

![图片](https://images-cdn.shimo.im/nVC6msJVdA86PYNJ/image.png!thumbnail)

## 代码参数详解

### channel通道


* channel方法详解：[rabbitmq channel参数详解](https://www.cnblogs.com/piaolingzxh/p/5448927.html)
* queueDeclare方法参数详解：[https://blog.csdn.net/vbirdbest/article/details/78670550](https://blog.csdn.net/vbirdbest/article/details/78670550)
```
channel.queueDeclare（）
channel.basicQos（）
channel.basicPublish（）
channel.basicAck（）
channel.basicConsume（）
```

* 为队列设置消息TTL

TTL 是 Time-To-Live 的缩写，指的是存活时间。RabbitMQ 可以为每一个队列设置其内部消息的 TTL。

```
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-message-ttl", 5000);
consumerChannel.queueDeclare(QUEUE_NAME, false, false, true, args);
```

* 为单条消息设置TTL

也可以为每一条消息设置存活时间。

```
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder().expiration("500").build();
senderChannel.basicPublish("", QUEUE_NAME, properties, message.getBytes("UTF-8"));
```

* 队列TTL

通过设置队列TTL，如果指定时间内队列没被使用，则队列自动被删除。

```
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-expires", 5000);
consumerChannel.queueDeclare(QUEUE_NAME, false, false, true, args);
```
args参数：

x-max-length 参数限制了一个队列的消息总数，当消息总数达到限定值时，队列头的消息会被抛弃。

x-max-length-bytes队列中消息体总字节数进行限制

x-dead-letter-exchange 死信队列

### Exchange类型


* [rabbitmq direct、fanout、topic 三种Exchange java 代码比较](https://www.cnblogs.com/piaolingzxh/p/5448757.html)
### 消息应答ack


* boolean autoAck = true;(自动确认模式)
    * 一旦 RabbitMQ 将消息分发给了消费者，就会从内存中删除。在这种情况下，如果杀死正在执行任务的消费者，会丢失正在处理的消息，也会丢失已经分发给这个消费者但尚未处理的消息。
* boolean autoAck = false; (手动确认模式)
    * 我们不想丢失任何任务，如果有一个消费者挂掉了，那么我们应该将分发给它的任务交付给另一个消费者去处理。 为了确保消息不会丢失，RabbitMQ 支持消息应答。消费者发送一个消息应答，告诉 RabbitMQ 这个消息已经接收并且处理完毕了。RabbitMQ 可以删除它了。
* 消息应答是默认打开的。也就是 boolean autoAck =false;
### 消息持久化

boolean durable = true;

channel.queueDeclare("test_queue_work", durable, false, false, null);

## 生产者发送消息（exchange交换机模式）

### **Direct类型（路由模式）**

将消息中的RoutingKey与该Exchange关联的所有Binding中的BindingKey进行比较，如果相等，则发送到该Binding对应的Queue中。

![图片](https://images-cdn.shimo.im/meChClL08dI5fQmd/image.png!thumbnail)

### **Fanout 类型（发布订阅模式）**

则会将消息发送给所有与该  Exchange 定义过  Binding 的所有  Queues 中去，其实是一种广播行为

![图片](https://images-cdn.shimo.im/2wtrH7Nehu4PoCq3/image.png!thumbnail)

### **Topic类型（通配符模式）**

则会按照正则表达式，对RoutingKey与BindingKey进行匹配，如果匹配成功，则发送到对应的Queue中

![图片](https://images-cdn.shimo.im/3C637kFOh1oCzfDR/image.png!thumbnail)

## 消息确认机制（事务+confirm）

在使用RabbitMQ的时候，我们可以通过消息持久化操作来解决因为服务器的异常奔溃导致的消息丢失，除此之外我们还会遇到一个问题，当消息的发布者在将消息发送出去之后，消息到底有没有正确到达broker代理服务器呢？如果不进行特殊配置的话，默认情况下发布操作是不会返回任何信息给生产者的，也就是默认情况下我们的生产者是不知道消息有没有正确到达broker的，如果在消息到达broker之前已经丢失的话，持久化操作也解决不了这个问题，因为消息根本就没到达代理服务器，你怎么进行持久化，那么这个问题该怎么解决呢？

RabbitMQ为我们提供了两种方式：


* （事务机制）通过AMQP事务机制实现，这也是AMQP协议层面提供的解决方案；
* （Confirm模式）通过将channel设置成confirm模式来实现；
### 1、AMQP事务机制

![图片](https://images-cdn.shimo.im/p3mrmWdWsfcdF71y/image.png!thumbnail)


* txSelect：用户将当前channel设置成transaction模式
* txCommit：用于提交事务
* txRollback：回滚事务

这里首先探讨下RabbitMQ事务机制。

RabbitMQ中与事务机制有关的方法有三个：txSelect(), txCommit()以及txRollback(), txSelect用于将当前channel设置成transaction模式，txCommit用于提交事务，txRollback用于回滚事务，在通过txSelect开启事务之后，我们便可以发布消息给broker代理服务器了，如果txCommit提交成功了，则消息一定到达了broker了，如果在txCommit执行之前broker异常崩溃或者由于其他原因抛出异常，这个时候我们便可以捕获异常通过txRollback回滚事务了。

事务机制的优缺点：

事务确实能够解决producer与broker之间消息确认的问题，只有消息成功被broker接受，事务提交才能成功，否则我们便可以在捕获异常进行事务回滚操作同时进行消息重发，但是使用事务机制的话会降低RabbitMQ的性能，那么有没有更好的方法既能保障producer知道消息已经正确送到，又能基本上不带来性能上的损失呢？从AMQP协议的层面看是没有更好的方法，但是RabbitMQ提供了一个更好的方案，即将channel信道设置成confirm模式。

### 2、Comfirm模式

生产者将信道设置成confirm模式，一旦信道进入confirm模式，所有在该信道上面发布的消息都会被指派一个唯一的ID(从1开始)，一旦消息被投递到所有匹配的队列之后，broker就会发送一个确认给生产者（包含消息的唯一ID）,这就使得生产者知道消息已经正确到达目的队列了，如果消息和队列是可持久化的，那么确认消息会将消息写入磁盘之后发出，broker回传给生产者的确认消息中deliver-tag域包含了确认消息的序列号，此外broker也可以设置basic.ack的multiple域，表示到这个序列号之前的所有消息都已经得到了处理。

confirm模式最大的好处在于他是异步的，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息，如果RabbitMQ因为自身内部错误导致消息丢失，就会发送一条nack消息，生产者应用程序同样可以在回调方法中处理该nack消息。

在channel 被设置成 confirm 模式之后，所有被 publish 的后续消息都将被 confirm（即 ack） 或者被nack一次。但是没有对消息被 confirm 的快慢做任何保证，并且同一条消息不会既被 confirm又被nack 。

核心代码:

```
//生产者通过调用channel的confirmSelect方法将channel设置为confirm模式
channel.confirmSelect();
```
1. 普通 confirm 模式：每发送一条消息后，调用waitForConfirms()方法，等待服务器端

confirm。实际上是一种串行 confirm 了。

2. 批量 confirm 模式：每发送一批消息后，调用 waitForConfirms()方法，等待服务器端

confirm。

3.异步 confirm 模式：提供一个回调方法，服务端 confirm 了一条或者多条消息后 Client 端会回

调这个方法


* 同步模式：
```
//使用waitForConfirms等待确认，串行模式（同步）
if(channel.waitForConfirms()) {
    System.out.println("消息发送成功~~");
} else {
    System.out.println("消息发送失败！！！");
}
```
* 异步模式（监听模式）

当一个消费者向RabbitMQ注册后，RabbitMQ会用basic.deliver 方法向消费者推送消息，这个方法携带了一个*delivery tag*， 它在一个channel中唯一代表了一次投递。delivery tag的唯一标识范围限于channel。

delivery tag是单调递增的正整数，客户端获取投递的方法用用dellivery tag作为一个参数

```
//通道监听
channel.addConfirmListener(new ConfirmListener() {
    //没问题的回调ack方法
    //每回调一次handleAck方法，unconfirm集合删掉相应的一条（multiple=false） 或多条（multiple=true）记录。
    public void handleAck(long deliveryTag, boolean multiple) throws IOException {
        System.out.println("正常回调--> deliveryTag-" + deliveryTag + "  multiple-" + multiple);
    }
    //有问题的回调nack方法，1s，3s，10s
    public void handleNack(long deliveryTag, boolean multiple) throws IOException {
        System.out.println("异常回调--> deliveryTag-" + deliveryTag + "  multiple-" + multiple);
    }
});
```
可以通过一下代码获取
```
long nextSeqNo = channel.getNextPublishSeqNo();
```
## 消费者订阅消息

通过basic.consume命令，订阅某一个队列中的消息,channel会自动在处理完上一条消息之后，接收下一条消息。（同一个channel消息处理是串行的）。除非关闭channel或者取消订阅，否则客户端将会一直接收队列的消息。

6

通过basic.get命令主动获取队列中的消息，但是绝对不可以通过循环调用basic.get来代替basic.consume，这是因为basic.get RabbitMQ在实际执行的时候，是首先consume某一个队列，然后检索第一条消息，然后再取消订阅。如果是高吞吐率的消费者，最好还是建议使用basic.consume。

## Springboot集成RabbitMq

## [2、RabbitMq安装与组件.xmind](https://uploader.shimo.im/f/qdU6Q06NQOoQnd0V.xmind)[3、五种队列模式.xmind](https://uploader.shimo.im/f/9vo4MwkRuSgJ6Nbo.xmind)[4、消息确认机制.xmind](https://uploader.shimo.im/f/R0NTDFCuBbwxFbVo.xmind)[5、消息应答与持久化.xmind](https://uploader.shimo.im/f/aES3oUlYo7EnHpHu.xmind)[课程18预习、消息中间件RabbitMq.xmind](https://uploader.shimo.im/f/hQe5u8giaC8CNz8Y.xmind)[1、RabbitMq简介.xmind](https://uploader.shimo.im/f/ddLZcK8YucseAbCL.xmind)

第一步，导入坐标

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```
第二步、配置rabbitmq的服务器信息
```
spring:
  rabbitmq:
    host: localhost
    port: 5672
```
第三步，声明交换机，路由键、队列等信息，方便监听
```
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class RabbitConfig {
    // 队列名称
    public final static String SPRING_BOOT_QUEUE = "spring-boot-queue-1";
    // 交换机名称
    public final static String SPRING_BOOT_EXCHANGE_queue = "spring-boot-topic-exchange-queue-1";
    //
    public final static String SPRING_BOOT_EXCHANGE = "spring-boot-topic-exchange-1";
    // 绑定的值
    public static final String SPRING_BOOT_BIND_KEY = "topic.message";

    @Bean
    public Queue helloQueue() {
        return new Queue(SPRING_BOOT_QUEUE);
    }
    //--------------------
    @Bean
    public Queue exQueue() {
        return new Queue(SPRING_BOOT_EXCHANGE_queue);
    }
    @Bean
    TopicExchange exchange() {
        return new TopicExchange(SPRING_BOOT_EXCHANGE);
    }
    @Bean
    Binding bindingExchangeMessage(Queue exQueue, TopicExchange exchange) {
        return BindingBuilder.bind(exQueue).to(exchange).with(SPRING_BOOT_BIND_KEY);
    }

}
```
第四步、监听队列信息
```
@Component
@RabbitListener(queues = RabbitConfig.SPRING_BOOT_QUEUE)
public class HelloReceiver {
    @RabbitHandler
    public void process(String hello) {
        System.out.println("You receiver a message : --> " + hello);
    }
}
```
第五步、发送信息
```
@RestController
public class MqController {
    @Autowired
    private AmqpTemplate amqpTemplate;
    @GetMapping("/send")
    public void send () {
        //往队列发送信息
        amqpTemplate.convertAndSend(RabbitConfig.SPRING_BOOT_QUEUE,"hello springboot rabbitmq");
        //往交换机发送信息
        amqpTemplate.convertAndSend(RabbitConfig.SPRING_BOOT_EXCHANGE, RabbitConfig.SPRING_BOOT_BIND_KEY, "hello springboot exchange message!!");
    }
}
```
