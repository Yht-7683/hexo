---
title: 网络编程框架t-io
date: 2020-09-14 15:06:04
tags: io
excerpt: 转载 16、网络编程框架t-io
---

## [课程16、百万即时通讯t-io（预习）.xmind](https://uploader.shimo.im/f/npkshxRrSBccEKXM.xmind)[1_t-io简介.xmind](https://uploader.shimo.im/f/PnfxSILbFNkD2WeU.xmind)[2_两个官方例子.xmind](https://uploader.shimo.im/f/mDqvLHxxEGoZZwiY.xmind)[课程16_百万即时通讯t-io_预习_.xmind](https://uploader.shimo.im/f/H086RZs57eYQNtHs.xmind)t-io是什么

![图片](https://images-cdn.shimo.im/Tw3bLi4pOR8bSM7W/image.png!thumbnail)

### 官网相关

官网：[https://t-io.org/](https://t-io.org/)

宣传：不仅仅是百万级网络通信框架，让天下没有难开发的网络通信

git地址：[https://gitee.com/tywo45/t-io](https://gitee.com/tywo45/t-io)

t-io手册：[https://t-io.org/doc/index.html](https://t-io.org/doc/index.html)

[t-io.pdf](https://attachments-cdn.shimo.im/6QSDGHEJpxkcJCwl/t_io.pdf)

## t-io与websocket

**t-io：**是一个网络框架，从这一点来说是有点像 netty 的，但 t-io 为常见和网络相关的业务（如 IM、消息推送、RPC、监控）提供了近乎于现成的解决方案，即丰富的编程 API，极大减少业务层的编程难度。

**websocket**：	WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工(full-duplex)通信——允许服务器主动发送信息给客户端。

t-io详细介绍：[https://blog.csdn.net/ningjiebing/article/details/90207332](https://blog.csdn.net/ningjiebing/article/details/90207332)

对半包和粘包的处理：源码分析见：[https://my.oschina.net/talenttan/blog/1610690](https://my.oschina.net/talenttan/blog/1610690)

## t-io框架

### 使用场景

![图片](https://images-cdn.shimo.im/65B2axTnX3kFyV3c/image.png!thumbnail)

### 常用关键类


* **ChannelContext(通道上下文)**
    * 每一个 tcp 连接的建立都会产生一个 ChannelContext 对象
    * （1）*ServerChannelContext*
        * ChannelContext 的子类，当用 tio 作 tcp 服务器时，业务层接触的是这个类的实例。
    * （2）*ClientChannelContext*
        * ChannelContext 的子类，当用 tio 作 tcp 客户端时，业务层接触的是这个类的实例

![图片](https://images-cdn.shimo.im/pBsf85g1uj05F8Lu/image.png!thumbnail)


* **GroupContext(服务配置与维护)**
    * GroupContext 就是用来配置线程池、确定监听端口，维护客户端各种数据等的
    * ClientGroupContext
    * ServerGroupContext

我们在写 TCP Server 时，都会先选好一个端口以监听客户端连接，再创建 N 组线程池来执行相关的任 务，譬如发送消息、解码数据包、处理数据包等任务，还要维护客户端连接的各种数据，为了和业务互动， 还要把这些客户端连接和各种业务数据绑定起来，譬如把某个客户端绑定到一个群组，绑定到一个 userid， 绑定到一个 token 等。GroupContext 就是用来配置线程池、确定监听端口，维护客户端各种数据等的。

GroupContext 是个抽象类，如果你是用 tio 作 tcp 客户端，那么你需要创建 ClientGroupContext，如 果你是用 tio 作 tcp 服务器，那么你需要创建 ServerGroupContext


* **AioHandler(消息处理接口)**
    * 处理消息的核心接口，它有两个子接口
    * ClientAioHandler
    * ServerAioHandler
* **AioListener(通道监听者)**
    * 处理事件监听的核心接口，它有两个子接口，
    * ClientAioListener
    * ServerAioListener
* **Packet(应用层数据包)**
    * TCP 层过来的数据，都会被 tio 要求解码成 Packet 对象，应用都需要继承这个类，从而实现自己的业务 数据包。

![图片](https://images-cdn.shimo.im/iDYMMY4vCg8IeFuj/image.png!thumbnail)

用于应用层与传输层的数据传递

![图片](https://images-cdn.shimo.im/kENWz1jWbeQe8CbC/image.png!thumbnail)

传输层在往应用层传递数据时，并不保证每次传递的数据是一个完整的应用层数据包（以 http 协议为 例，就是并不保证应用层收到的数据刚好可以组成一个 http 包），这就是我们经常提到的**半包**和**粘包**。传输层只负责传递 byte[]数据，应用层需要自己对 byte[]数据进行解码，以 http 协议为例，就是把 byte[] 解码成 http 协议格式的字符串。


* AioServer（tio 服务端入口类）

![图片](https://images-cdn.shimo.im/qrWTMXfeZC01dgll/image.png!thumbnail)


* AioClient（tio 客户端入口类）

![图片](https://images-cdn.shimo.im/rIdwLOTRwCgnbqJE/image.png!thumbnail)


* **ObjWithLock（自带读写锁的对象）**
    * 是一个自带了一把（读写）锁的普通对象（一般是集合对象），每当要对 这个对象进行同步安全操作（并发下对集合进行遍历或对集合对象进行元素修改删除增加）时，就得用这个 锁。

***t-io是基于tcp层协议的一个网络框架，所以在应用层与tcp传输层之间设计到一个数据的编码与解码问题，t-io让我们能自定义数据协议，所以需要我们自己手动去编码解码过程。***

## 入门例子HelloWord


* git项目地址

[https://gitee.com/tywo45/tio-showcase](https://gitee.com/tywo45/tio-showcase)

git里面有个几个例子，首先我们看helloword的例子：

### 业务逻辑

本例子演示的是一个典型的TCP长连接应用，大体业务简介如下。


* 分为server和client工程，server和client共用common工程
* 服务端和客户端的消息协议比较简单，消息头为4个字节，用以表示消息体的长度，消息体为一个字符串的byte[]
* 服务端先启动，监听6789端口
* 客户端连接到服务端后，会主动向服务器发送一条消息
* 服务器收到消息后会回应一条消息
* 之后，框架层会自动从客户端发心跳到服务器，服务器也会检测心跳有没有超时（这些事都是框架做的，业务层只需要配一个心跳超时参数即可）
* 框架层会在断链后自动重连（这些事都是框架做的，业务层只需要配一个重连配置对象即可）

具体类说明：

### server端


* **导入核心包**
```
<dependency>
   <groupId>org.t-io</groupId>
   <artifactId>tio-core</artifactId>
</dependency>
```
* **HelloServerStarter**

构造ServerGroupContext，main方法启动服务


* **HelloServerAioHandler**

实现ServerAioHandler接口，重写decode，encode，handler方法。

### common**端**


* **HelloPacket**

继承Packet类。自定义数据包的内容


* **Const**

常量类，配置ip，端口等信息

### client端


* **HelloClientStarter**

构造clientGroupContext，连接节点，使用信息发送消息


* **HelloClientAioHandler**

实现ClientAioHandler接口，重写decode，encode，handler方法。

### 流程图


* *idea请安装****PlantUML intergration****插件*

[客户端与服务端沟通时序图.puml](https://attachments-cdn.shimo.im/eexjhhIVSuENPmSO/客户端与服务端沟通时序图.puml)[Server端初始化时序图.puml](https://attachments-cdn.shimo.im/jXf6fJfb4FgTS94e/Server端初始化时序图.puml)

![图片](https://images-cdn.shimo.im/OMayDEX45DcnSLbd/image.png!thumbnail)

**（初始化服务器）**


![图片](https://images-cdn.shimo.im/yaB4Ezz9JEMaC6Bn/image.png!thumbnail)

**（客户端与服务端通讯流程）**

## 入门例子showcase


* git项目地址

[https://gitee.com/tywo45/tio-showcase](https://gitee.com/tywo45/tio-showcase)

上面讲的是helloword的例子，比较简单，接下来的是showcase的例子，结合实际场景的一个例子。

### 业务逻辑

tio的框架初始化使用过程是一样的。不过因为showcase中涉及到的交互越来越多，因此不能像helloword例子中的HelloPacket只有body这么简单了，我们至少要加上一个type参数（消息类型），这样的话服务器获取到数据包之后再更加type来选择消息处理类，从而拓展系统可用性。

![图片](https://images-cdn.shimo.im/oNdIg15oi3sw58WA/image.png!thumbnail)

（客户端与服务器沟通流程）

[客户端与服务端沟通时序图.puml](https://attachments-cdn.shimo.im/drAAqMvA9akpRi85/客户端与服务端沟通时序图.puml)

客户端发起登录操作


* org.tio.examples.showcase.client.**ShowcaseClientStarter**#processCommand
```
LoginReqBody loginReqBody = new LoginReqBody();
loginReqBody.setLoginname(loginname);
loginReqBody.setPassword(password);
ShowcasePacket reqPacket = new ShowcasePacket();
#这里指定消息类型
reqPacket.setType(Type.LOGIN_REQ);
reqPacket.setBody(Json.toJson(loginReqBody).getBytes(ShowcasePacket.CHARSET));
Tio.send(clientChannelContext, reqPacket);
```
* **LoginReqBody**：登录请求参数封装类，继承BaseBody
* **ShowcasePacket**：数据包，继承Packet，和ByteBuffer相互转换
* **clientChannelContext**：连接上下文通道

服务端接受请求操作


* org.tio.examples.showcase.server.**ShowcaseServerAioHandler**#handler
```
ShowcasePacket showcasePacket = (ShowcasePacket) packet;
#获取消息类型
Byte type = showcasePacket.getType();
#根据消息类型找到对应的消息处理类
AbsShowcaseBsHandler<?> showcaseBsHandler = handlerMap.get(type);
if (showcaseBsHandler == null) {
   log.error("{}, 找不到处理类，type:{}", channelContext, type);
   return;
}
#执行消息处理。消息处理类必须继承AbsShowcaseBsHandler
showcaseBsHandler.handler(showcasePacket, channelContext);
```
* **handlerMap**：存有当前所有消息处理类的map。数据包中包含消息类型，会根据消息类型获取对应的消息处理类，而这个消息处理类会调用handler（）方法处理数据。
* **AbsShowcaseBsHandler**：消息处理抽象类，继承这个类的处理类会对一种消息类型进行处理，并且一般专门处理一种消息封装类（继承BaseBody的封装类）。

消息类型对应消息处理类的初始化

```
private static Map<Byte, AbsShowcaseBsHandler<?>> handlerMap = new HashMap<>();
static {
   #把消息类型与消息处理类映射起来 
   handlerMap.put(Type.GROUP_MSG_REQ, new GroupMsgReqHandler());
   handlerMap.put(Type.HEART_BEAT_REQ, new HeartbeatReqHandler());
   handlerMap.put(Type.JOIN_GROUP_REQ, new JoinGroupReqHandler());
   handlerMap.put(Type.LOGIN_REQ, new LoginReqHandler());
   handlerMap.put(Type.P2P_REQ, new P2PReqHandler());
}
```
如果接收到的消息类型是P2P_REQ，那么处理类就是P2PReqHandler：


* org.tio.examples.showcase.server.handler.**P2PReqHandler**#handler
```
log.info("收到点对点请求消息:{}", Json.toJson(bsBody));
ShowcaseSessionContext showcaseSessionContext = (ShowcaseSessionContext) channelContext.getAttribute();
P2PRespBody p2pRespBody = new P2PRespBody();
p2pRespBody.setFromUserid(showcaseSessionContext.getUserid());
p2pRespBody.setText(bsBody.getText());
ShowcasePacket respPacket = new ShowcasePacket();
respPacket.setType(Type.P2P_RESP);
respPacket.setBody(Json.toJson(p2pRespBody).getBytes(ShowcasePacket.CHARSET));
Tio.sendToUser(channelContext.groupContext, bsBody.getToUserid(), respPacket);
```
## springboot集成t-io

项目运行：[https://github.com/fanpan26/SpringBootLayIM.git](https://github.com/fanpan26/SpringBootLayIM.git)

由于layim是付费产品，所以在网络上找了一个。（仅供学习哈）

需要放到项目目录下面

static/js/layui/lay/modules/layim.js

[layim.js](https://attachments-cdn.shimo.im/1CrKWGWmd3k6AVQ3/layim.js)

### 项目结构

*项目集成：*

因为这里需要和浏览器之间进行通讯，所以需要用到websocket机制。tio有集成websocket的框架，所以直接导入即可。

```
<dependency>
    <groupId>org.t-io</groupId>
    <artifactId>tio-websocket-server</artifactId>
    <version>0.0.5-tio-websocket</version>
</dependency>
```
*代码结构*
![图片](https://images-cdn.shimo.im/BLrbVsdNvPg719v7/image.png!thumbnail)

### Guava - EventBus(事件总线)

项目使用Guava的EventBus替代了Spring的ApplicationEvent事件机制。

Guava的EventBus使用介绍如下：

**事件定义**

EventBus为我们提供了register方法来订阅事件，不需要实现任何的额外接口或者base类，只需要在订阅方法上标注上**@Subscribe**和保证**只有一个输入参数**的方法就可以搞定。

```
new Object() {
    @Subscribe
    public void lister(Integer integer) {
        System.out.printf("%d from int%n", integer);
    }
}
```
**事件发布**

对于事件源，则可以通过post方法发布事件。 正在这里对于Guava对于事件的发布，是依据上例中订阅方法的方法参数类型决定的，换而言之就是post传入的类型和其基类类型可以收到此事件。

```
//定义事件
final EventBus eventBus = new EventBus();
//注册事件
eventBus.register(new Object() {
    //使用@Subscribe说明订阅事件处理方法
    @Subscribe
    public void lister(Integer integer) {
        System.out.printf("%s from int%n", integer);
    }
    @Subscribe
    public void lister(Number integer) {
        System.out.printf("%s from Number%n", integer);
    }
    @Subscribe
    public void lister(Long integer) {
        System.out.printf("%s from long%n", integer);
    }
});
//发布事件
eventBus.post(1);
eventBus.post(1L);
```
***项目的而运用***

**主要处理事件**：包含了申请通知，添加好友成功通知

**关键类：**


* com.fyp.layim.common.event.bus.**EventUtil**：封装了事件的监听注册，以及发布动作
* com.fyp.layim.common.event.bus.body.**EventBody**：发布的内容封装类，包含消息类型和消息内容字段
* com.fyp.layim.common.event.bus.handler.**AbsEventHandler：**事件处理抽象类，具体处理器需要继承这个重写handler（）方法
* com.fyp.layim.common.event.bus.handler.**AddFriendEventHandler：**添加好友成功通知处理类。
* com.fyp.layim.common.event.bus.**LayimEventType：**消息类型常量

**调用：**


* com.fyp.layim.web.biz.**UserController**#handleFriendApply：好友同意好友请求之后发布事件

**逻辑：**

事件的处理其实是给申请人发起好友同意通知


* com.fyp.layim.im.common.util.**PushUtil**#pushAddFriendMessage
```
/**
 * 添加好友成功之后向对方推送消息
 * */
public static void pushAddFriendMessage(long applyid){
    if(applyid==0){
        return;
    }
    Apply apply = applyService.getApply(applyid);
    ChannelContext channelContext = getChannelContext(""+apply.getUid());
    //先判断是否在线，再去查询数据库，减少查询次数
    if (channelContext != null && !channelContext.isClosed()) {
        LayimToClientAddFriendMsgBody body = new LayimToClientAddFriendMsgBody();
        User user = getUserService().getUser(apply.getToid());
        if (user==null){return;}
        //对方分组ID
        body.setGroupid(apply.getGroup());
        //当前用户的基本信息，用于调用layim.addList
        body.setAvatar(user.getAvatar());
        body.setId(user.getId());
        body.setSign(user.getSign());
        body.setType("friend");
        body.setUsername(user.getUserName());
        push(channelContext, body);
    }
}
```
最后是通过Aio.send发送消息。

* com.fyp.layim.im.common.util.**PushUtil**#push
```
/**
 * 服务端主动推送消息
 * */
private static void push(ChannelContext channelContext,Object msg) {
    try {
        WsResponse response = BodyConvert.getInstance().convertToTextResponse(msg);
        Aio.send(channelContext, response);
    }catch (IOException ex){
    }
}
```
## 功能结构


* **登录功能**
* **单聊功能**
* **群聊功能**
* **其他自定义消息提醒功能**
* **等等。。。。**

登录的目的是过滤非法请求，如果有一个非法用户请求websocket服务，直接返回403或者401即可。

单聊，群聊这个就不用解释了

其他自定义消息提醒，比如：时时加好友消息，广播消息，审核消息等。

## 项目设计-前端

前端的框架用到是layim，layim已经帮我们做好了界面和封装了接口，所以我们的工作只需要按照layim来写好接口提供结果就可以了。

首先看初始化：


* templates/index.html

![图片](https://images-cdn.shimo.im/wyHKOJjA4LY5x4Jl/image.png!thumbnail)

```
layim.config({
    //初始化接口
    init: {
        url: '/layim/base'
    }
    //查看群员接口
    ,members: {
        url: '/layim/members'
    }
    //上传图片接口
    ,uploadImage: {url: '/upload/file'}
    //上传文件接口
    ,uploadFile: {url: '/upload/file'}
    ,isAudio: true //开启聊天工具栏音频
    ,isVideo: true //开启聊天工具栏视频
    ,initSkin: '5.jpg' //1-5 设置初始背景
    ,notice: true //是否开启桌面消息提醒，默认false
    ,msgbox: '/layim/msgbox'
    ,find: layui.cache.dir + 'css/modules/layim/html/find.html' //发现页面地址，若不开启，剔除该项即可
    ,chatLog: layui.cache.dir + 'css/modules/layim/html/chatLog.html' //聊天记录页面地址，若不开启，剔除该项即可
});
```
因为需要进行通讯，所以websocket的信息也需要初始化。都是在layim里面一起初始化的。
```
socket.config({
    log:true,
    token:'/layim/token',
    server:'ws://127.0.0.1:8888'
});
```

* templates/layim/msgbox.html
## 项目设计-后端

### 基本相关类

控制器：

com.fyp.layim.web.api.**GroupController：**群组相关

com.fyp.layim.web.biz.AccountController：账户相关，登录注册

com.fyp.layim.web.biz.UploadController：上传文件、图片

com.fyp.layim.web.biz.UserController：用户基础数据

拦截器：

com.fyp.layim.web.filter.TokenVerifyInterceptor：校验token，与userid转换

com.fyp.layim.web.filter.LayimAspect：给所有请求设置当前用户的值，放session当中

全局异常处理：

com.fyp.layim.web.filter.GlobalExceptionHandler：异常处理

### 网络编程主要代码

com.fyp.layim.im.common.*：定义消息类型，消息处理类的公共包。

com.fyp.layim.im.packet.*：这是数据包，因为协议是websocket，所以返回的不是packet，最后要返回的其实是符合websocket定义的org.tio.websocket.common.WsResponse。而这些数据是放在body中。

com.fyp.layim.im.server.*：t-io的服务配置相关，包括t-io服务的启动，启动之前需要ServerAioHandler、ServerAioListener、ServerGroupContext等参数。

所以总体逻辑是这样的：

步骤一、配置了springboot的模块自动加载机制。


* META-INF/spring.factories
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration= com.fyp.layim.im.server.LayimServerAutoConfig
```
LayimServerAutoConfig配置将会被自动装载。


* com.fyp.layim.im.server.LayimServerAutoConfig：自动装载的配置类
* com.fyp.layim.im.server.config.LayimServerConfig：服务的ip、端口、心跳时间等基本配置
* com.fyp.layim.im.server.LayimWebsocketStarter：初始化配置，启动t-io服务，其中配置初始化和启动都是委托给com.fyp.layim.im.server.LayimServerStarter完成。
* com.fyp.layim.im.server.LayimServerStarter：根据配置初始化serverGroupContext、初始化消息处理器
```
//初始化t-io的serverGroupContext
//还有消息处理器与消息类型的映射关系
public LayimServerStarter(LayimServerConfig wsServerConfig, IWsMsgHandler wsMsgHandler, TioUuid tioUuid, SynThreadPoolExecutor tioExecutor, ThreadPoolExecutor groupExecutor) throws Exception {
    this.layimServerConfig = wsServerConfig;
    this.wsMsgHandler = wsMsgHandler;
    layimServerAioHandler = new LayimServerAioHandler(wsServerConfig, wsMsgHandler);
    layimServerAioListener = new LayimServerAioListener();
    serverGroupContext = new ServerGroupContext(layimServerAioHandler, layimServerAioListener, tioExecutor, groupExecutor);
    //心跳时间，暂时设置为0
    serverGroupContext.setHeartbeatTimeout(wsServerConfig.getHeartBeatTimeout());
    serverGroupContext.setName("Tio Websocket Server for LayIM");
    aioServer = new AioServer(serverGroupContext);
    serverGroupContext.setTioUuid(tioUuid);
    //initSsl(serverGroupContext);
    
    
    //初始化消息处理器
    LayimMsgProcessorManager.init();
}
```
接下来看下与serverGroupContext相关的消息处理咧、事件监听类

* com.fyp.layim.im.server.handler.LayimServerAioHandler：继承ServerAioHandler，完成消息的解码、编码、消息处理过程
* com.fyp.layim.im.server.listener.LayimServerAioListener：继承ServerAioListener，完成事件监听

绑定用户信息


* com.fyp.layim.im.common.processor.**ClientCheckOnlineMsgProcessor**#process
```
SetWithLock<ChannelContext> checkChannelContexts =
      Aio.getChannelContextsByUserid(channelContext.getGroupContext(),body.getId());
```
绑定用户上线信息。


* com.fyp.layim.im.server.handler.**LayimMsgHandler**#handleHandshakeUserInfo
```
private HttpResponse handleHandshakeUserInfo(HttpRequest httpRequest, HttpResponse httpResponse, ChannelContext channelContext) throws  Exception {
    UserService userService = getUserService();
    //增加token验证方法
    String path = httpRequest.getRequestLine().getPath();
    String token = URLDecoder.decode(path.substring(1),"utf-8");
    String userId = TokenVerify.IsValid(token);
    if (userId == null) {
        //没有token 未授权
        httpResponse.setStatus(HttpResponseStatus.C401);
    } else {
        long uid = Long.parseLong(userId);
        //解析token
        LayimContextUserInfo userInfo = userService.getContextUserInfo(uid);
        if (userInfo == null) {
            //没有找到用户
            httpResponse.setStatus(HttpResponseStatus.C404);
        } else {
            channelContext.setAttribute(userId, userInfo.getContextUser());
            //绑定用户ID
            Aio.bindUser(channelContext, userId);
            //绑定用户群组
            List<String> groupIds = userInfo.getGroupIds();
            //绑定用户群信息
            if (groupIds != null) {
                groupIds.forEach(groupId -> Aio.bindGroup(channelContext, groupId));
            }
            //通知所有好友本人上线了
            notify(channelContext,true);
        }
    }
    return httpResponse;
}
```

Aio.java的api：

![图片](https://images-cdn.shimo.im/G2W3fERzQLAMRNvZ/image.png!thumbnail)

实现**IWsMsgHandler**接口：用于websocket握手，响应，关闭通道等过程的业务处理。

![图片](https://images-cdn.shimo.im/GwpqDOvqBHkYpBMo/image.png!thumbnail)

![图片](https://images-cdn.shimo.im/JmJxBpZYWF8CMGxV/image.png!thumbnail)

握手逻辑首先是在：com.fyp.layim.im.server.handler.LayimServerAioHandler中。

wsMsgHandler.handshake 方法，这里一般直接返回默认的 httpReponse即可，代表（框架层）握手成功。但是我们可以在接口中自定义一些业务逻辑，比如用户判断之类的逻辑，然后决定是否同意握手流程。

![图片](https://images-cdn.shimo.im/DcgHQBSvsA8yr1Do/image.png!thumbnail)

![图片](https://images-cdn.shimo.im/gXRfryi52x0t2J1h/image.png!thumbnail)

