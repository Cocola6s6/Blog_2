---
title: 笔记-物联网IOT
categories: 好的学习笔记
tags: IOT
---

* 设备接入方案-1
* 设备接入方案-2
* MQTT协议接入
* 共享电动车定位方案



# 设备接入方案-1

## 一、数据结构

### 1、协议格式

没有通用的协议格式，根据不同设备不同的协议而定。



### 2、消息格式

所有的设备消息都会被解析转换为对应的消息对象，然后在服务器之间传递。

~~~bash
## 属性、遥测、服务、事件消息
struct {
	propertiesData: List,
	telemetryData: List,
	serviceData: List,
	eventData: List,
}
~~~



## 二、指令下发



## 三、消息上报

流程如下：

1. 收到 netty 的消息。
2. 解码器 Encoder，将消息解码为设备对应的消息对象。
3. 通过自定义 nettyHandler。
4. 通过消息处理器，找到消息转换器。根据 json 配置文件，将消息对象转换为按照【 属性、遥测、服务、事件】分类的消息对象。
5. 消息分发到消息队列，消息队列分发消息到不通的业务服务。
6. 规则引擎服务将消息分发到 Distruptor。
7. Distruptor 收到消息，按顺序对消息进行过滤、持久化、通知等业务处理。



如下图所示。

![image-20231220171439531](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231220171439531.png)



## 四、会话管理

流程如下：

1. 从 channel 拿 sessionInfo，如果没有证明是第一次上报，此时初步创建会话并将会话保持到 channel 全局常量中。会话信息只包含一些大概信息，如 channel、ip、crateTime 等。
2. 认证消息，对比数据库对消息进行认证，如果通过了，则保持会话到 globalSession 中。此时会话信息包含了详细信息，如 productId、devId、lastActiveTime、isAuth 等。



数据结构如下所示。

~~~java
// value是sessionInfo
public static AttributeKey<Session> SESSION = AttributeKey.valueOf("device.session");
// value是true/false
public static final AttributeKey<Boolean> reLease = AttributeKey.valueOf("downLine.reLease");


// 保存通过认证的会话
protected Map<Long /**deviceId **/, T> globalSession = new ConcurrentHashMap();
~~~







# 设备接入方案-2

## 一、数据结构

### 1、协议格式

~~~bash
// 通用协议格式
struct ProtocolBase {
	msgType: int,
	devId: String,
	txnNo: long,
}

// 蓝牙的协议格式-1
struct BlueToothExchangeProtocol {
	srcList: List<ExChangeDetail>,
}
// 蓝牙的协议格式-2
struct ExChangeDetail {
	txnNo: long,
	userId: String,
	emptyBoxBt: String,
	emptyBoxId: int,
	fullBoxBt: String,
	fullBoxId: int,
	result: int,
	stopCode: String,
	extVal: Strng,
}

// ....................其他
~~~



### 2、消息格式

所有的设备消息都会被解析转换为对应的消息对象，然后在服务器之间传递。

~~~bash
struct EventInfo {
	eventType: EventType,	// 事件类型
	protocoType: int,	// 协议类型
	message: ProtocolBase,	// 协议格式
	channel: channel,	// netty channel
}
~~~



### 3、支持的柜子的事件类型

事件类型对应了 Disruptor 的消费者。

~~~bash
enum EventType {
    /**
     * 客户端注册
     */
    CLIENT_REGISTER,

    /**
     * 心跳检查
     */
    HEART_BEAT,

    /**
     * 客户端发过来的协议包
     */
    CLIENT_PROTO_COMMING,

    /**
     * 客户端发过来的协议包
     */
    CLIENT_NONIMPORTANT_PROTO_COMMING,

    /**
     * 客户端关闭连接
     */
    CLIENT_DISCONNECT,

    /**
     * 下发命令
     */
    SEND_COMMAND,

    /**
     * 客户端定时发过来的协议包
     * 1.蓝牙检查
     */
    CLIENT_PROTO_INTERVAL_COMMING
}
~~~



### 4、协议类型

每个协议类型对应自己的处理器。

~~~bash
struct ProtocolType {
	request: RequestConstants
	response: ResponseConstants
}

enum RequestConstants {
	RUNTIME_VALUE(310), // 属性上报请求
	ALARM_VALUE(410),	// 告警上报请求
	CONTROL_VALUE(500),	// 控制下发请求
}

enum ResponseConstants {
	RUNTIME_VALUE(301), // 属性上报响应
	ALARM_VALUE(411),	// 告警上报响应
	CONTROL_VALUE(501),	// 控制下发响应
}
~~~



## 二、指令下发

流程如下：

1. 收到 rabbitMq 的消息。
2. 组装 Disrutpor 的消息：eventType:SEND_COMMAND, protocoType: BLUE_TOOTH_CHECK。通过 Disrutpor 分发消息。
3. Disrutpor 消费者收到消息，根据事件类型找到对应的处理方法。
4. 通过 onSendCommand()，向 netty channel 写入消息。
5. 编码器 DefaultEncoder，将消息编码为字节。





## 三、消息上报

流程如下：

1. 收到 netty 的消息。
2. 解码器 Encoder，将消息解码为设备消息对象 EventInfo。
3. 通过 eventType，分发到不用的 Disrutpor。
4. Disrutpor 消费者收到消息，根据 eventType 找到对应的事件处理器。
5. 通过 protocoType ，找到对应的协议处理器。
6. 协议处理器将消息发送到 rabbitMq，到其他业务系统进行业务处理。



如下图所示。

![image-20231220171453679](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231220171453679.png)



## 四、会话管理

会话的数据结构，如下所示。

~~~java
// 上行时使用，通过chnanel找到具体的设备信息
private Map<Channel, SessionInfo> sessions = new HashMap<>();
// 下行时使用，通过设备ID查找到对应的channel对象
private Map<String, Channel> channels = new HashMap<>();

private static final String lastSessionTimeKey = "lastSessionTime";
private static final String loginSuccessKey = "loginSuccess";
~~~



流程如下：

1. 第一次消息上报，进入自定义 onClientRegisterHandler。主要的操作就是，保存当前会话到 sessions 中。
2. 客户端心跳上报，进入自定义 closeTimeoutSessionHandler。主要操作就是，根据 lastSessionTimeKey、loginSuccessKey 判断客户端是否是超时了，然后 从 sessions 中移除。



【注意】lastSessionTimeKey 和 loginSuccessKey 的作用

这两个 key 是通过 channel 的 AttributeMap 管理的，相当于它们是在 channel 的全局共享常量。在每次收到上行消息时，都会更新 lastSessionTimeKey。在收到登录成功消息时，就更新 loginSuccessKey，标识客户端已经登录。



【问题】为什么不把这个 lastSessionTimeKey，loginSuccessKey 都放到 sessionInfo 里呢？然后直接将 sessionInfo 设置为 channel 全局共享常量

* 难道说，是为了节省带宽？我觉得应该是最开始设计应该考虑不够。loginSuccessKey 是后续





【问题】为什么会话管理创建了两个 map 进行维护，感觉都是一个东西？

理解一下这个问题的意思，如下：

~~~java
// 因为 devId-->channel, channel-->sessionInfo
// 所以 devId-->sessionInfo
private Map<Long deviceId, SessionInfo> globalSession = new ConcurrentHashMap();
~~~





【问题】为什么是通过客户端心跳上报来删除超时的连接？而不是服务端主动轮询删除？

* 和服务器压力有关。减轻了服务器压力，如果服务器需要定期轮询所有连接，那么当连接数量非常多的时候，这会导致线程一直被占用，服务器压力增大。换成客户端主动通知的方式，在有需要的时候再执行，大大减轻了服务器压力。
* 和实时性有关。客户端心跳上报，服务器就可以实时知道这个连接的状态了，而不需要等待下一次轮询。

这有没有一种，【回调函数机制】的味儿在里面，类似 epoll 的异步 IO 的原理，从主动轮询变成完成后得到通知，大大提高了 CPU 资源利用率。参考【Redis笔记-I/O 多路复用模型】





## 五、详细设计

### 1、事件类型对应了 Disruptor 的消费者

~~~java
public DefaultQueueConsumer() {
    eventHandlers.put(EventType.CLIENT_REGISTER, this::onClientRegister);
	eventHandlers.put(EventType.CLIENT_PROTO_COMMING, this::onClientProtoCome);
	eventHandlers.put(EventType.CLIENT_DISCONNECT, this::onClientDisconnect);
	eventHandlers.put(EventType.SEND_COMMAND, this::onSendCommand);
	eventHandlers.put(EventType.HEART_BEAT, this::closeTimeoutSession);
}
~~~



设备的上行、下行消息都通过 Disruptor 进行分发，还有客户端和服务端通信的消息也通过 Disruptor 进行分发通过。可以看出它定义了五个消费者：

* 注册
* 断开连接
* 超时连接
* 上行消息处理
* 下行消息处理



对比方案1，相同的是。它们的客户端和服务端的通信的操作【注册、断开连接、超时连接】，都是在 netty 的 handler 处理的。

* 方案1，使用的是 channelRead()、channelInactive() 方法。并且直接处理。
* 方案2，使用的是 channelRegistered()、channelInactive()、extractObject() 方法。不是直接处理，是通过 Disruptor 转发消息。

方案1 的注册没有使用 channelRegistered，而是在 channelRead 中进行业务判断后处理。



【问题】我不理解为什么它所有的消息都进行了分发处理？除了 extractObject() 以外，其它的方法只会执行一次或者频率不高，分发是为了异步处理使得当前 channel 的任务快速完成，这样子线程就可以被分配到其它任务。 

* 方案2只使用了 decoder、encoder 这两个 ChannelHandler，并且只是对消息进行了 byte 转为 String，然后就交由后续的 disruptor 来处理。
* 为了减少 netty 中的 ChannelHandler 的占用时间，腾出更多的时间来处理其他客户端。这也是能让一个微服务处理更多连接的基础。



### 2、协议类型对应了设备支持的所有操作

~~~java
public void registerProtocolHandler() {
    handlers.put(ProtocolType.LOGIN, loginProtocolHandler);
    handlers.put(ProtocolType.ALARM, alramProtocolHandler);
    handlers.put(ProtocolType.CONFIG_REPORT, configReportProtocolHandler);
    handlers.put(ProtocolType.ResponseConstants.CONTROL_VALUE, controlProtocolHandler);
    handlers.put(ProtocolType.ResponseConstants.CONFIG_SEARCH_VALUE, configSearchProtocolHandler);
    handlers.put(ProtocolType.ResponseConstants.CONFIG_VALUE, configReplyProtocolHandler);
    handlers.put(ProtocolType.BLUE_TOOTH, blueToothExChangeProtocolHandler);
}
~~~



### 3、会话管理



## 六、总结

### 1、netty 的使用

* 方案1，用到了自定义编码解码器和自定义处理器

* 方案2，只用到了自定义编码解码器，解码完成直接就是分发到业务处理。

  

方案2这样做是为了减少对 netty 的占用，把 netty 的压力给到 Distruptor。



### 2、Distruptor 的使用

方案1，主要用它来对【 属性、遥测、服务、事件】消息进行过滤，因为这些是大数据量。对比如下：

* 如果使用单线程，服务器资源就那么多，如果 ruleEngineService 服务链路长，那么 MQ 消息就会发生积压。
* 如果使用多线程，某个时间段消息的数量非常多时，线程池数量不够用，会发生锁等待，某个时间段吞吐量不够。
* 使用 Distruptor，无锁设计，线程级别的“异步”处理。消息队列是服务间的异步。



方案2，就是把它当“消息队列”的作用使用了。收到设备消息就往 Distruptor 丢，然后让 Distruptor 根据消息类型走不同的业务处理。

【问题】为什么它不增加一个 MQ 来对设备消息进行一个削峰、解构呢？

* 方案2，它只有一款设备，不需要过度设计。
* 方案1，可是十几款设备。





### 3、下行数据处理

* 方案1 : kafka/Restful --> 自定义 rpc，收到指令后进行解析到对应的 rpc 进行处理。
* 方案2: rabbitMq --> disruptor，收到消息队列里的指令后直接给到 disruptor 处理。

方案1，是用 kafka 和 Restful 来处理下行数据的。因为它涉及的下行数据更为复杂。而方案2，指令只有“一步”。方案1，它是支持多协议的，并且还需要进行许多业务处理。如判断“指令是否需要同步处理”，“指令是否需要自旋处理”等。



### 4、上行数据处理

* 方案1：设备消息 --> Kafka --> Disruptor --> Akka --> DB，
* 方案2：设备消息 --> Kafka --> Apache Storm(Disruptor) --> DB

很类似，方案1，其实就是使用【Disruptor、Akka】实现的一套自定义的分布式计算的框架，因为 Apache Storm  本身也可以集成 Disruptor 来提高消息处理的性能。在 Storm 中使用 Disruptor 可以作为其内部的数据传递机制。Kafka 是需要计算的内容。Akka 是一个用于构建高并发和分布式应用程序的并发编程工具包。









# MQTT协议接入

[MQTT 协议是个啥？](https://www.cnblogs.com/cxuanBlog/p/14917187.html)



## 为什么要使用 MQTT 协议？

### 1、发布 - 订阅模式

pub/sub 最重要的方面是 publisher 与 subscriber 的解藕，这种耦合度有下面三个维度：

- 空间解耦：publisher 与 subscriber 并不知道对方的存在，例如不会有 IP 地址和端口的交互，也更不会有消息的交互。
- 时间解藕：publisher 与 subscriber 并不一定需要同时运行。
- 同步 `Synchronization` 解藕：两个组件的操作比如 publish 和 subscribe 都不会在发布或者接收过程中产生中断。

总之，发布/订阅模式消除了传统客户-服务器之间的直接通信，把通信这个操作交给了 broker 进行代理，并在空间、时间、同步三个维度上进行了解藕。



### 2、MQTT 与消息队列的区别

我们现在知道，MQTT 是一种消息队列传输探测协议，这种协议是看似是以消息队列为基础，但却与消息队列有所差别。

* 在消息队列中，不会存在消息没有客户端消费的情况。但是在 MQTT 中，存在 topic 无 subscriber 订阅的情况。

* 在消息队列模式中，一条消息只能被一个客户端所消费。而在 MQTT 中，每个订阅者都会受到消息，每个订阅者有相同的负载。

  > 多个消费者绑定同一个消息队列是无法同时消费的，一个消息只能被一个消费者消费一个消息只能被一个消费者消费。负载均衡机制会进行消息的轮询分发。

* 在消息队列模式中，必须使用单独的命令来显式创建队列；而在 MQTT 中，topic 比较灵活，可以即时创建。

> MQTT Topic 非常轻量级，client 在发布或订阅之前不需要先创建所需要的 Topic，broker 在接收每个 Topic 前不用进行初始化操作。



## 使用 Netty 应用 MQTT 

### 1、Client

实现客户端是为了方便进行测试和调试。

![image-20230301115322395](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230301115322395.png)



![image-20230301115723985](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230301115723985.png)



### 2、Server

![image-20230301115356646](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230301115356646.png)



![image-20230301115749000](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230301115749000.png)



![image-20230301120029793](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230301120029793.png)







# 共享电动车定位方案

## 一、前提

怎么确定电动车是否停放在停放区？电动车停车的时候，需要判断用户是否停放在停放区。



## 二、整体设计

### 1、停放区

【问题】停放区是多边形的，怎么存储多边形？

* 使用 pgsql 的地理数据类型 geography。
* pgsql 还提供了专用的空间函数和运算符，进行空间数据的查询、分析和可视化

![image-20240319170649676](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240319170649676.png)



数据结构如下：

~~~rust
struct FenceBean {
    // 围栏类型
    fenceType: String,
    // 城市id
    cityId: int,
    // 围栏中心点wkt格式
    centerPointWkt: String,
    // 围栏wkt格式
    pointsWkt: String,
}
~~~



数据库结构如下：

~~~sql
CREATE TABLE "public"."fence_tb" (
  "code" int8 NOT NULL DEFAULT nextval('fence_tb_code_seq'::regclass),
  "fence_type" varchar(32) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
  "name" varchar(64) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
  "city_id" int8 NOT NULL DEFAULT 0,
  "center_point" "public"."geography" NOT NULL,
  "points" "public"."geography" NOT NULL,
  "address" varchar(128) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
  "fence_status" int2 NOT NULL DEFAULT 0,
  "update_user_id" int4 NOT NULL DEFAULT 0,
  "create_user_id" int4 NOT NULL DEFAULT 0,
  "create_user_name" varchar(32) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
  "update_user_name" varchar(32) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
  "remark" varchar(255) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
  "deleted" int2 NOT NULL DEFAULT 0,
  "create_time" timestamp(6),
  "update_time" timestamp(6),
  "type" varchar(32) COLLATE "pg_catalog"."default" DEFAULT ''::character varying,
  "vertical_parking_start_point" varchar(32) COLLATE "pg_catalog"."default" DEFAULT ''::character varying,
  "vertical_parking_end_point" varchar(32) COLLATE "pg_catalog"."default" DEFAULT ''::character varying,
  "vertical_parking_angle" float4 DEFAULT '-1'::integer,
  CONSTRAINT "fence_tb_pkey" PRIMARY KEY ("code")
)
;
~~~



### 2、车辆位置

位置信息可以通过下发指令给设备获得，也可以通过设备主动上报的消息获得。建议是通过设备上报的方式。一是更加实时性，因为下发指令可能有延时。二是更加可靠，消息上报是多次的，下发指令还不能保证正常响应。

设备上报的车辆位置消息，消息经过 Kafka，然后在经过分布式计算框架 Flink 计算，最后写入 Redis。



## 三、详细设计

### 1、怎么在 pgsql 中存储停放区

入参是围栏的所有坐标点集合，处理是需要将坐标集合转换为 pgsql 的 geography 类型。

1. 获取坐标集合
2. 转换为 wkt 格式 ploygon
3. 插入 pg



简单的 insertSql 写法如下所示。

~~~sql
INSERT INTO fence_tb (polygon_column) VALUES ('POLYGON((x1 y1, x2 y2, x3 y3, x4 y4, x1 y1))'::geography);
~~~

所以，需要进行如下转换。

~~~rust
// 坐标点集合, {{114.0, 22.0}, {114.1, 22.1}, {114.2, 22.2}}
struct request {
    points[][]: double,
}

转换为

// pgsql, POLYGON ((114.0 22.0, 114.1 22.1, 114.2 22.2, 114.0 22.0))
POLYGON((x1 y1, x2 y2, x3 y3, x4 y4, x1 y1))
~~~

最终转换如下所示。

![image-20240320095733438](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240320095733438.png)



最终 sql 如下所示。

![image-20240320095820927](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240320095820927.png)



### 2、如何查询车辆附近的停放区？

入参是车辆当前坐标，处理是需要将坐标点转换为 pgsql 的 point 类型。

1. 获取坐标
2. 转换为 wkt 格式 point
3. 通过 pg 的函数计算 point 到所有多边形的距离



简单的 querySql 写法如下所示。

* st_distance，可以计算两个几何体之间的距离
* st_astext，可以将转换为 wkt 格式
* st_dwithin，可以查询的半径范围。

~~~sql
select st_astext(points) as pointsWkt, st_distance(points, (POINT(x, y)::geography) as distance
from fence_tb
where st_dwithin(points, (POINT(x, y)::geography, #{radius})
order by distance
~~~



【问题】st_distance 的原理是什么，是两个多边形的中心之间的距离吗？

![image-20240319175132079](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240319175132079.png)



所以，需要进行如下转换。

~~~rust
// 当前坐标, "10,20"
struct request {
    point: String,
}

转换为

// pgsql, POINT (10.0 20.0)
POINT (x y)
~~~

最终转换如下所示。



![image-20240320100006436](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240320100006436.png)



最终 sql 如下所示。

![image-20240320095549738](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240320095549738.png)



### 3、如何判断车辆正确停放？

1. 停车是否在停放区
2. 判定是否垂直停车

是否在停放区，其实就是通过判断车辆和附近停放区的距离，如果距离为 0 则表示正确停放。

是否垂直停车，是想判断车辆有没有放正，斜着停放太多禁止停车。两个入参，车的方位和停放区的方位。如下所示。

```
车辆偏向角度为15度，垂直停车角度为10度, 偏差角度为10度, 可停车角度为[0, 20] 和 [180, 200], [0, 20]是正着停放，[180, 200] 是倒着停放
```





### 4、如何查询车辆附近的停放区？（实时）

前面是主动询问，通过 pgsql 的历史上报数据判断。如果需要实时获取车辆的运行位置情况，需要在消息上报的时候，通过 Flink 实时计算。如下图所示。

1. 拿到位置 point
2. 通过 point 计算距离多边形的距离。
3. 距离大于 0 则表示不在停放区。



![image-20240320110702679](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240320110702679.png)