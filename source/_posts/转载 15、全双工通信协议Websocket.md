---
title: 全双工通信协议Websocket
date: 2020-09-14 15:04:04
tags: Websocket
excerpt: 转载 15、全双工通信协议Websocket
---


## [4_spirngboot集成websocket.xmind](https://uploader.shimo.im/f/UqXxQuoImL88n4n9.xmind)[15预习、即时通讯websocket.xmind](https://uploader.shimo.im/f/p2iXdytafMEUhYTh.xmind)[15预习_即时通讯websocket.xmind](https://uploader.shimo.im/f/EPjOwRHwuHIA5OSL.xmind)[1_websocket简介.xmind](https://uploader.shimo.im/f/t6tKKOib0PktU0it.xmind)[2_websocket握手过程.xmind](https://uploader.shimo.im/f/J6WicHe02UgAzwuJ.xmind)[3_java_WebSocket实现.xmind](https://uploader.shimo.im/f/WvC5Qcf6bV0m1ctW.xmind)百度百科：websocket

WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工(full-duplex)通信——允许服务器主动发送信息给客户端。

WebSocket使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在WebSocket API中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

## http的不足，websocket的出现

### websocket背景

了解计算机网络协议的人，应该都知道：HTTP 协议是一种无状态的、无连接的、单向的应用层协议。它采用了请求/响应模型。通信请求只能由客户端发起，服务端对请求做出应答处理。

这种通信模型有一个弊端：HTTP 协议无法实现服务器主动向客户端发起消息。

这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦。大多数 Web 应用程序将通过频繁的异步JavaScript和XML（AJAX）请求实现长轮询。轮询的效率低，非常浪费资源（因为必须不停连接，或者 HTTP 连接始终打开）。

因此，工程师们一直在思考，有没有更好的方法。WebSocket 就是这样发明的。WebSocket 连接允许客户端和服务器之间进行全双工通信，以便任一方都可以通过建立的连接将数据推送到另一端。WebSocket 只需要建立一次连接，就可以一直保持连接状态。这相比于轮询方式的不停建立连接显然效率要大大提高。

![图片](https://images-cdn.shimo.im/TNgrgWYa5icIWn2b/image.png!thumbnail)

http解决双工常用方法：

长久以来, 创建实现客户端和用户端之间双工通讯的web app都会造成HTTP轮询的滥用: 客户端向主机不断发送不同的HTTP呼叫来进行询问。

### 桥梁技术 --ajax轮询

ajax轮询 的原理非常简单，让浏览器隔个几秒就发送一次请求，询问服务器是否有新信息。

场景再现：

客户端：啦啦啦，有没有新信息(Request)

服务端：没有（Response）

客户端：啦啦啦，有没有新信息(Request)

服务端：没有。。（Response）

客户端：啦啦啦，有没有新信息(Request)

服务端：你好烦啊，没有啊。。（Response）

客户端：啦啦啦，有没有新消息（Request）

服务端：好啦好啦，有啦给你。（Response）

客户端：啦啦啦，有没有新消息（Request）

服务端：。。。。。没。。。。没。。。没有（Response） ---- loop

### 桥梁技术 -- 长轮询

long poll 其实原理跟 ajax轮询 差不多，都是采用轮询的方式，不过采取的是阻塞模型（一直打电话，没收到就不挂电话），也就是说，客户端发起连接后，如果没消息，就一直不返回Response给客户端。直到有消息才返回，返回完之后，客户端再次建立连接，周而复始。

场景再现

客户端：啦啦啦，有没有新信息，没有的话就等有了才返回给我吧（Request）

服务端：额。。 等待到有消息的时候。。来 给你（Response）

客户端：啦啦啦，有没有新信息，没有的话就等有了才返回给我吧（Request） -loop

## websocket特点

![图片](https://images-cdn.shimo.im/vclkrQtHdYMqcZbb/image.png!thumbnail)


1. 服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种。
2. 建立在 TCP 协议之上，服务器端的实现比较容易。
3. 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
4. 数据格式比较轻量，性能开销小，通信高效。
5. 可以发送文本，也可以发送二进制数据。
6. 没有同源限制，客户端可以与任意服务器通信。
7. 协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。
```
ws://example.com:80/some/path
```
## websocket实现原理

在实现websocket连线过程中，需要通过浏览器发出websocket连线请求，然后服务器发出回应，这个过程通常称为“握手” 。在 WebSocket API，浏览器和服务器只需要做一个握手的动作，然后，浏览器和服务器之间就形成了一条快速通道。两者之间就直接可以数据互相传送。在此WebSocket 协议中，为我们实现即时服务带来了两大好处：


* 1. Header

互相沟通的Header是很小的-大概只有 2 Bytes


* 2. Server Push

服务器的推送，服务器不再被动的接收到浏览器的请求之后才返回数据，而是在有新数据时就主动推送给浏览器。

## websocket与http的关系

![图片](https://images-cdn.shimo.im/DZ8SmLG3rV8QimIZ/image.png!thumbnail)

首先Websocket是基于HTTP协议的，或者说借用了HTTP的协议来完成一部分握手。

我们来看个典型的 Websocket 握手（借用Wikipedia的。。）

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
```
熟悉HTTP的童鞋可能发现了，这段类似HTTP协议的握手请求中，多了几个东西。我会顺便讲解下作用。
```
Upgrade: websocket
Connection: Upgrade
```
这个就是Websocket的核心了，告诉 Apache 、 Nginx 等服务器：注意啦，我发起的是Websocket协议，快点帮我找到对应的助理处理~不是那个老土的HTTP。
```
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```
首先， Sec-WebSocket-Key 是一个 Base64 encode 的值，这个是浏览器随机生成的，告诉服务器：泥煤，不要忽悠窝，我要验证尼是不是真的是Websocket助理。与后面服务端响应首部的Sec-WebSocket-Accept是配套的，提供基本的防护，比如恶意的连接，或者无意的连接。
然后， Sec_WebSocket-Protocol 是一个用户定义的字符串，用来区分同URL下，不同的服务所需要的协议。简单理解：今晚我要服务A，别搞错啦~

最后， Sec-WebSocket-Version 是告诉服务器所使用的 Websocket Draft（协议版本），在最初的时候，Websocket协议还在 Draft 阶段，各种奇奇怪怪的协议都有，而且还有很多期奇奇怪怪不同的东西，什么Firefox和Chrome用的不是一个版本之类的，当初Websocket协议太多可是一个大难题。。不过现在还好，已经定下来啦~大家都使用的一个东西~ 脱水： 服务员，我要的是13岁的噢→_→

然后服务器会返回下列东西，表示已经接受到请求， 成功建立Websocket啦！

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```
这里开始就是HTTP最后负责的区域了，告诉客户，我已经成功切换协议啦~
```
Upgrade: websocket
Connection: Upgrade
```
依然是固定的，告诉客户端即将升级的是 Websocket 协议，而不是mozillasocket，lurnarsocket或者shitsocket。
然后， Sec-WebSocket-Accept 这个则是经过服务器确认，并且加密过后的 Sec-WebSocket-Key 。

Sec-WebSocket-Accept 的计算方法：


* 将 Sec-WebSocket-Key 跟 258EAFA5-E914-47DA-95CA-C5AB0DC85B11 拼接；
* 通过 SHA1 计算出摘要，并转成 base64 字符串。

作用：


1. 避免服务端收到非法的websocket连接（比如http客户端不小心请求连接websocket服务，此时服务端可以直接拒绝连接）
2. 确保服务端理解websocket连接。因为ws握手阶段采用的是http协议，因此可能ws连接是被一个http服务器处理并返回的，此时客户端可以通过Sec-WebSocket-Key来确保服务端认识ws协议。（并非百分百保险，比如总是存在那么些无聊的http服务器，光处理Sec-WebSocket-Key，但并没有实现ws协议。。。）
3. Sec-WebSocket-Key主要目的并不是确保数据的安全性，因为Sec-WebSocket-Key、Sec-WebSocket-Accept的转换计算公式是公开的，而且非常简单，最主要的作用是预防一些常见的意外情况（非故意的）。

后面的， Sec-WebSocket-Protocol 则是表示最终使用的协议。

至此，HTTP已经完成它所有工作了，接下来就是完全按照Websocket协议进行了。

连接成功的状态码是101。

![图片](https://images-cdn.shimo.im/TPyXUxTUDnEYzbhy/image.png!thumbnail)

## java WebSocket实现


* 可参考：[https://www.jianshu.com/p/d79bf8174196](https://www.jianshu.com/p/d79bf8174196)

在maven中添加websocket库的代码如下:

```
<dependency> 
  <groupId>javax.websocket</groupId> 
  <artifactId>javax.websocket-api</artifactId> 
  <version>1.1</version>
  <scope>provided</scope>
</dependency>
```
### **注解、成员数据介绍**

**@ServerEndpoint**

**声明websocket地址**类似Spring MVC中的@controller注解类似，websocket使用@ServerEndpoint来进行声明接口：@ServerEndpoint(value="/websocket/{paraName}") ; 其中 “ { } ”用来表示带参数的连接，如果需要获取{}中的参数在参数列表中增加：@PathParam("paraName") Integer userId 。

**1.@OnOpen**

public void onOpen(Session session) throws IOException{ }-------有连接时的触发函数。 我们可以在用户连接时记录用户的连接带的参数，只需在参数列表中增加参数：@PathParam("paraName") String paraName。

**2.@OnClose**

public void onClose(){ }------连接关闭时的调用方法。

**3.@OnMessage**

public void onMessage(String message, Session session) { }-------收到消息时调用的函数，其中Session是每个websocket特有的数据成员

**4.Session**----每个Session代表了两个web socket断点的会话；当websocket握手成功后，websocket就会提供一个打开的Session，可以通过这个Session来对另一个端点发送数据；如果Session关闭后发送数据将会报错。

**5.Session.getBasicRemote().sendText("message")**-------向该Session连接的用户发送字符串数据。

**6.@OnError**

public void onError(Session session, Throwable error) { }--------发生意外错误时调用的函数。

***websocket demo git：***


* [https://gitee.com/lv-success/git-second/tree/master/course-17-websocket/websocketDemo](https://gitee.com/lv-success/git-second/tree/master/course-17-websocket/websocketDemo)
## springboot + websocket项目实现

Websocket 是通过一个socket来实现双工异步通讯的能力。但是直接使用WebSocket协议开发程序显得特别烦琐，我门会使用它的子协议STOMP，它是一个更高级级别的协议，STOMP协议使用一个基于帧（frame）的格式来定义信息。

### SockJS

正如我们所知,websocket协议虽然已经被制定，当时还有很多版本的浏览器或浏览器厂商还没有支持的很好。

所以,SockJS,可以理解为是websocket的一个备选方案。

那它如何规定备选方案的呢？

它大概支持这样几个方案:


* Websockets
* Streaming
* Polling

当然，开启并使用SockJS后，它会优先选用websocket协议作为传输协议，如果浏览器不支持websocket协议，则会在其他方案中，选择一个较好的协议进行通讯。

![图片](https://images-cdn.shimo.im/vYQCjSEgLIw3JdmP/image.png!thumbnail)

此图来源于 github: sockjs-client

所以，如果使用SockJS进行通讯，它将在使用上保持一致，底层由它自己去选择相应的协议。

可以认为SockJS是websocket通讯层上的上层协议。底层对于开发者来说是透明的。

### STOMP

STOMP 中文为: 面向消息的简单文本协议

websocket定义了两种传输信息类型:**文本信息 和 二进制信息 ( text and binary )**。类型虽然被确定，但是他们的传输体是没有规定的。

当然你可以自己来写传输体，来规定传输内容。(当然，这样的复杂度是很高的)

所以,需要用一种简单的文本传输类型来规定传输内容，它可以作为通讯中的文本传输协议,即交互中的高级协议来定义交互信息。

STOMP本身可以支持流类型的网络传输协议: websocket协议和tcp协议。

stomp是一个用于client之间进行异步消息传输的简单文gg本协议, 全称是Simple Text Oriented Messaging Protocol.

对于stomp协议来说, client分为消费者client与生产者client两种. server是指broker, 也就是消息队列的管理者.

stomp协议并不是为websocket设计的, 它是属于消息队列的一种协议, 和amqp, jms平级.

只不过由于它的简单性恰巧可以用于定义websocket的消息体格式.

stomp协议很多mq都已支持, 比如rabbitmq, activemq. 很多语言也都有stomp协议的解析client库.

可以这么理解, websocket结合stomp相当于一个面向公网对用户比较友好的一种消息队列.

stomp协议中的client分为两角色:


* 生产者: 通过SEND命令给某个目的地址(destination)发送消息.
* 消费者: 通过SUBSCRIBE命令订阅某个目的地址(destination), 当生产者发送消息到目的地址后, 订阅此目的地址的消费者会即时收到消息.

它的格式为:

![图片](https://images-cdn.shimo.im/QQQlY39xE0ob0Hlv/image.png!thumbnail)

## springboot 基于子协议STOMP开发的websocket

### 后端技术方案选型

websocket服务端选型:spring websocket

支持SockJS,开启SockJS后，可应对不同浏览器的通讯支持

支持STOMP传输协议，可无缝对接STOMP协议下的消息代理器(如:RabbitMQ, ActiveMQ)

### 前端技术方案选型

前端选型: stomp.js,sockjs.js

后端开启SOMP和SockJS支持后，前对应有对应的js库进行支持.

所以选用此两个库.

### 技术选型总结

上述所用技术，是这样的逻辑:


* **开启socktJS:**

如果有浏览器不支持websocket协议，可以在其他两种协议中进行选择，但是对于应用层来讲，使用起来是一样的。

这是为了支持浏览器不支持websocket协议的一种备选方案


* **使用STOMP:**

使用STOMP进行交互，前端可以使用stomp.js类库进行交互，消息一STOMP协议格式进行传输，这样就规定了消息传输格式。

消息进入后端以后，可以将消息与实现STOMP格式的代理器进行整合。

这是为了消息统一管理，进行机器扩容时，可进行负载均衡部署


* **使用spring websocket:**

使用spring websocket,是因为他提供了STOMP的传输自协议的同时，还提供了StockJS的支持。

当然，除此之外，spring websocket还提供了权限整合的功能，还有自带天生与spring家族等相关框架进行无缝整合。

### websocket信息流

![图片](https://images-cdn.shimo.im/NQJtZrlKtNsAbrlc/image.png!thumbnail)


* Message在应用中的流动是这样一个流程，如上图。若destination是以/app开始则会通过request channel交给注解方法来处理，处理完毕根据默认的路径转发给SimpleBroker处理（若不使用默认路径可以用@SendTo来指定路径），处理完毕后交由response channel返回连接的客户端。
* 若destination是以/topic开头则直接交给SimpleBroker处理。

**第一步**：导入spirngboot与websocket集成的pom坐标

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
<!--添加jsp支持-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <scope>provided</scope>
</dependency>
```
**第二步**：自定义websocket配置，配置内容包括开启子协议STOMP，配置服务端点，前缀，等信息。
```
@Configuration
@EnableWebSocketMessageBroker//注解表示开启使用STOMP协议来传输基于代理的消息，Broker就是代理的意思。
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    /***
     * 注册 Stomp的端点
     * addEndpoint：添加STOMP协议的端点。提供WebSocket或SockJS客户端访问的地址
     * withSockJS：使用SockJS协议
     * @param registry
     */
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/endpointWisely")
                .withSockJS() ;
        registry.addEndpoint("/websocket")
                .setAllowedOrigins("*")//添加允许跨域访问
                .withSockJS() ;
    }
    /**
     * 配置消息代理
     * 启动Broker，消息的发送的地址符合配置的前缀来的消息才发送到这个broker
     */
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/api/v1/socket/send","/user/", "/topic");//推送消息前缀
        registry.setApplicationDestinationPrefixes("/api/v1/socket/req");//应用请求前缀
        registry.setUserDestinationPrefix("/user");//推送用户前缀
    }
}
```
Spring框架提供基于websocket的STOMP支持，需要使用spring-messaging和spring-websocket模块。
下面的配置中，注册了一个前缀为/endpointWisely的stomp终端，客户端可以使用该url来建立websocket连接。

Message的destination如果是以/app开头，则会转发给响应的消息处理方法（如使用@MessageMapping注解的方法）,

如果是以/topic,/queue开头则会被转发给消息代理（broker），由broker广播给连接的客户端。

![图片](https://images-cdn.shimo.im/Rukwp3E7ZyY522wT/image.png!thumbnail)

**第三步**：上面配置完了之后，就可以开始编写内容了。这里表示服务器发送地址映射到/welcomeTopic，然后所有订阅了/topic/getResponse路径的都可以收到广播消息。

```
@MessageMapping("/welcomeTopic")//浏览器发送请求通过@messageMapping 映射/welcome 这个地址。
@SendTo("/topic/getResponse")//服务器端有消息时,会订阅@SendTo 中的路径的浏览器发送消息。
public ResponseMessage say(RequestMessage message) throws Exception {
    System.out.println("发送信息-----------------------" + message.getMessage());
    return new ResponseMessage("Welcome, " + message.getMessage() + "!");
}
```
第四步：由于使用了sockjs，因此前端需要导入相关的js文件。

[sockjs.js](https://attachments-cdn.shimo.im/m26ikyinB4Qfo3JO/sockjs.js)[stomp.js](https://attachments-cdn.shimo.im/AqgaaeZQ6ngHLnwb/stomp.js)[jquery.js](https://attachments-cdn.shimo.im/Y82sNuyzzas97dXb/jquery.js)

页面如下：

[topic.jsp](https://attachments-cdn.shimo.im/ngtor3hDy6UNvDsI/topic.jsp)

### 常用注解


* @EnableWebSocketMessageBroker

通过EnableWebSocketMessageBroker 开启使用STOMP协议来传输基于代理(message broker)的消息,此时浏览器支持使用@MessageMapping 就像支持@RequestMapping一样。


* @MessageMapping

配置中定义的config.setApplicationDestinationPrefixes("/app");表示如果链接以/app开头，则会转发给对应具有@MessageMapping对应链接的注解方法处理。如链接是/app/welcome则会找到@MessageMapping("/welcome")注解对应的方法。


* @SendTo

可以把消息广播到路径上去，例如上面可以把消息广播到”/topic/greetings”,如果客户端在这个路径订阅消息，则可以接收到消息。


* @SendToUser

消息目的地有UserDestinationMessageHandler来处理，会将消息路由到发送者对应的目的地。默认该注解前缀为/user。如：用户订阅/user/hi ，在@SendToUser('/hi')查找目的地时，会将目的地的转化为/user/{name}/hi, 这个name就是principal的name值，该操作是认为用户登录并且授权认证，使用principal的name作为目的地标识。发给消息来源的那个用户。（就是谁请求给谁，不会发给所有用户，区分就是依照principal-name来区分的)。

spring websocket基于注解的@SendTo和@SendToUser虽然方便，但是有局限性，例如我这样子的需求，我想手动的把消息推送给某个人，或者特定一组人，怎么办，@SendTo只能推送给所有人，@SendToUser只能推送给请求消息的那个人，这时，我们可以利用SimpMessagingTemplate这个类。

SimpMessagingTemplate有俩个推送的方法


1. convertAndSend(destination, payload); //将消息广播到特定订阅路径中，类似@SendTo
2. convertAndSendToUser(user, destination, payload);//将消息推送到固定的用户订阅路径中，类似@SendToUser

* @DestinationVariable

这个注解用于动态监听路径，很想rest中的@PathVariable：

```
@MessageMapping("/queue/chat/{uid}")
public void chat(@Payload @Validated Message message, @DestinationVariable("uid") String uid, Principal principal) {
    String msg = "发送人: " + principal.getName() + " chat ";
    simpMessagingTemplate.convertAndSendToUser(uid,"/queue/chat",msg);
}
```
### 通讯层设计 – 1-1 && 1-n

**1-n topic:**

此方式，上述消息模型章节已经讲过，此处不再赘述

**1-1 queue:**

客服-用户沟通为1-1用户交互的案例

前端:

```
stompClient.subscribe('/user/queue/chat',function(greeting){
    showGreeting(greeting.body);
});
```
后端:
```
@MessageMapping("/queue/chat/{uid}")
public void chat(@Payload @Validated Message message, @DestinationVariable("uid") String uid, Principal principal) {
    String msg = "发送人: " + principal.getName() + " chat ";
    simpMessagingTemplate.convertAndSendToUser(uid,"/queue/chat",msg);
}
```
发送端:
```
function chat(uid) {
    stompClient.send("/app/queue/chat/"+uid,{},JSON.stringify({'title':'hello','content':'message content'}));
}
```
上述的转化，看上去没有topic那样1-n的广播要流畅，因为代码中采用约定的方式进行开发，当然这是由spring约定的。
