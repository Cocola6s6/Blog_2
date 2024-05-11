---
title: 笔记-Java
categories: 好的学习笔记
tags: Java
---



# Java 注意

### 1、Map 的 get() 容易发生空指针

get 的类型和 set 的类型不一样，int 用 Integer 都不行



### 2、对象new 完成以后，已经在堆中分配了空间，只是所有属性值都为 null。

判断为 null，Objects.isNull 返回 true



### 3、List 中的检验失效

需要增加 @Valid 注解

![image-20230329102600273](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102600273.png)



### 4、stream排序不生效的原因：

> steam 流是生成新的

![image-20230329102616452](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102616452.png)

### 5、使用 Collections.emptyList() 导致数据不可修改

![image-20230329102633802](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102633802.png)


![image-20230329102646830](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102646830.png)



### 6、Object.equel() 

> 比较的时候注意类型要一样

![image-20230329102706033](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102706033.png)



### 7、请求参数过多

下图，为了防止循环内调用，于是声明了批量查询。

![image-20231212110920481](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231212110920481.png)





如下调用，列表的时候是正常的，但是当复用做导出的时候，userIdList 就会过大，这时候接口会失败”参数异常“，因为是 post 还是 get 请求，参数都是有长度限制的。这时候需要做分段查询。

![image-20231212111122553](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231212111122553.png)



![image-20231212111404531](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231212111404531.png)





### 8、Java 查询优化

> 注意：有条件的查询，一般都需要查询所有，被删除的也要

![image-20230329102426025](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102426025.png)





### 9、contains 空指针异常

contains 如果元素是 null 的话，会抛出空指针异常。如下所示。

![image-20240202155404739](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240202155404739.png)

需要判空处理。如下所示。

![image-20240202155520261](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240202155520261.png)





# Java 悲观锁

### 1、for update

如果查询条件用了索引（无论是索引什么情况）/主键，那么 select … for update 就会进行行锁
如果是普通字段(没有索引/主键)，那么 select … for update 就会进行锁表。



### 2、synchronized

Java里面的同步原语synchronized关键字的实现也是悲观锁。





# Java StringBuffer与 StringBuilder

> 如果是在单线程下操作大量数据，应优先使用 StringBuilder 类；如果是在多线程下操作大量数据，应优先使用 StringBuilder 类。

* StringBuilder是线程不安全的，StringBuffer 是线程安全的。
* 如果只是在单线程中使用字符串缓冲区，则 StringBuilder 的效率会高些，但是当多线程访问时，最好使用 StringBuffer。
* 在执行效率方面，StringBuilder  > StringBuffer > String





![image-20230107171539860](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230107171539860.png)



# Java 多线程和资源

> 多线程是为了同步完成多项任务，不是为了提高运行效率，而是为了提高资源使用效率来提高系统的效率。



如下所示，消息队列消费消息时，需要访问数据库，数据库资源是固定的，大量任务需要访问该资源时，都分配一个线程去使用数据库资源就提高了资源的利用率。



![image-20230306163247444](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230306163247444.png)



![image-20230306163409454](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230306163409454.png)



![image-20230306163319948](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230306163319948.png)





# Java @Async

## 一、前提

我们在使用多线程的时候，往往需要创建 Thread 类，或者实现 Runnable 接口，如果要使用到线程池，我们还需要来创建 Executors。建议使用 ThreadPoolExecutor 创建线程池。

在使用 Spring 中，已经给我们做了很好的支持。只要要 @EnableAsync 就可以使用多线程。使用 @Async 就可以定义一个线程任务。但是它使用的是 ThreadPoolTaskExecutor 创建线程池。



## 二、ThreadPoolExecutor 和 ThreadPoolTaskExecutor 

ThreadPoolExecutor 是 JDK 线程池类。ThreadPoolTaskExecutor 是 Spring 对 ThreadPoolExecutor 封装后提供的线程池类。



【问题】Spring 为什么要封装得到一个新的线程池类？

* ThreadPoolTaskExecutor 继承自 AsyncTaskExecutor、SchedulingTaskExecutor 等，表示它支持了 Spring 的异步特性、定时任务特性。
* 将线程池交给 Spring 管理声明周期，什么时候创建、什么时候销毁。





【问题】为什么我要将线程池交给 Spring 管理呢？

当你在类中生成静态的线程池，我们知道，静态变量的初始化是在类加载时进行的，它们的生命周期一直持续到 JVM 销毁该类，这个过程是你不好控制的。所以，当我们使用 bean 注入的方式时，Spring 会帮我们管理线程池的生成和销毁。



如下所示。可以在 @Async 中指定 ThreadPoolTaskExecutor 线程池。

~~~java
@Bean
public ThreadPoolTaskExecutor ccsThreadPool() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(CORE_POOL_SIZE);
        executor.setMaxPoolSize(MAX_POOL_SIZE);
        executor.setQueueCapacity(QUEUE_CAPACITY);
        executor.setThreadNamePrefix(THREAD_NAME);
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
}


// 支持异步特性
@Async("ccsThreadPool")
public testAsync() {}


// Spring 管理声明周期
@Autowired
private ThreadPoolTaskExecutor ccsThreadPool;
~~~





如下图所示。

![image-20240124171443717](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240124171443717.png)





![image-20240124171531475](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240124171531475.png)



## 三、@Async 失效的情况

1. 异步方法使用 static 修饰
2. 异步类没有使用 @Component 注解（或其他注解）导致 Spring 无法扫描到异步类
3. 异步方法不能与异步方法在同一个类中
4. 类中需要使用 @Autowired 或 @Resource 等注解自动注入，不能自己手动 new 对象
5. 如果使用 SpringBoot 框架必须在启动类中增加 @EnableAsync 注解
6. 在 Async 方法上标注 @Transactional 是没用的。 在 Async 方法调用的方法上标注 @Transactional 有效。
7. 【调用被 @Async 标记的方法的调用者不能和被调用的方法在同一类中不然不会起作用！】
8. 【使用 @Async 时要求是不能有返回值的不然会报错的，返回值必须用包装类接收。】



【问题】为什么调用被 @Async 标记的方法的调用者不能和被调用的方法在同一类中不然不会起作用？

* 它们的原理都是动态代理，也就是说只有通过【Proxy】去调用方法才能够使方法有"增强“的效果



【问题】怎么获取代理类呢？

* 通过注解自动注入

* 通过上下文中获取：AsyncAopService service = (AsyncAopService) AopContext.currentProxy()

  



## 四、@Async 返回值

代码中采用异步调用，【AOP】 做来一层切面处理，底层是通过 JDK 动态代理实现，它返回值必须是包装类型。

> 代理的方法返回值类型为 void，就 return null；否则就 return method.invoke(my, args)。





## 五、@Async 的弊端

@Async 使用 spring 提供的默认线程池执行异步任务，默认线程池的配置如下，所以很明显，在无界队列和 Integer 最大值加持下，很可能会发生内存溢出的情况。

* 核心线程数：默认为1
* 最大线程数：默认为 Integer.MAX_VALUE
* 线程存活时间：默认为 60s
* **无界队列**





# Java Future

### 前提

【问题】Future 是什么?

* 多线程编程里继承 Thread 和 Runnable 接口都获取不到执行结果，而 Callable 接口配合 Future 可以接收多线程的处理结果



### Future 适用场景

![image-20230329101928670](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329101928670.png)



# Java ListenableFuture

**前提：**

> Guava 定义了 ListenableFuture 接口并继承了 JDK concurrent 包下的 Future 接口, 在高并发并且需要大量 Future 对象的情况下，推荐尽量使用 ListenableFuture 来代替

ListenableFuture 允许你注册回调方法(callbacks)，在运算（多线程执行）完成的时候进行调用, 或者在运算（多线程执行）完成后立即执行。这样简单的改进，使得可以明显的支持更多的操作，这样的功能在JDK concurrent中的Future是不支持的。



![image-20230329102024220](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102024220.png)



**例子1：**

![image-20230329102100293](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102100293.png)



**例子2：**

![image-20230329102118552](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102118552.png)



# Java 延时任务

### 前提

> [延迟任务总结](https://www.cnblogs.com/haoxinyue/p/6663720.html)

**背景：**

延迟任务有别于定式任务，定式任务往往是固定周期的，有明确的触发时间。而延迟任务一般没有固定的开始时间，它常常是由一个事件触发的，而在这个事件触发之后的一段时间内触发另一个事件。

延迟任务相关的业务场景如下：

* 物联网系统经常会遇到向终端下发命令，如果命令一段时间没有应答，就需要设置成超时。订单下单之后30分钟后，如果用户没有付钱，则系统自动取消订单。



### 1、数据库轮询

这是比较常见的一种方式，所有的订单或者所有的命令一般都会存储在数据库中。我们会起一个线程去扫数据库或者一个数据库定时Job。

![image-20230210121546784](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230210121546784.png)



### 2、JDK ScheduledExecutorService

> JDK自带的一种线程池，它能调度一些命令在一段时间之后执行，或者周期性的执行



![image-20230210121851727](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230210121851727.png)



### 3、Quartz

> quartz是一个企业级的开源的任务调度框架，quartz内部使用TreeSet来保存Trigger，如下图。Java中的TreeSet是使用TreeMap实现，TreeMap是一个红黑树实现。
>
> 相比上述的两种轻量级的实现功能丰富很多。有专门的任务调度线程，和任务执行线程池。quartz功能强大，主要是用来执行周期性的任务，当然也可以用来实现延迟任务。但是如果只是实现一个简单的基于内存的延时任务的话，quartz就稍显庞大。



![image-20230210124612414](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230210124612414.png)



### 4、Jesque

> Jesque是Resque的java实现，Resque是一个基于Redis的Ruby项目，用于后台的定时任务。Jesque实现延迟任务的方法也是在Redis里面建立一个ZSet。
>
> Redis中的ZSet是一个有序的Set，内部使用HashMap和跳表(SkipList)来保证数据的存储和有序



**使用Redis的好处主要是：**

1. 解耦：把任务、任务发起者、任务执行者的三者分开，逻辑更加清晰，程序强壮性提升，有利于任务发起者和执行者各自迭代，适合多人协作。
2. 异常恢复：由于使用Redis作为消息通道，消息都存储在Redis中。如果发送程序或者任务处理程序挂了，重启之后，还有重新处理数据的可能性。
3. 分布式：如果数据量较大，程序执行时间比较长，我们可以针对任务发起者和任务执行者进行分布式部署。特别注意任务的执行者，也就是Redis的接收方需要考虑分布式锁的问题。



![image-20230210125031810](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230210125031810.png)



### 5、RabbitMQ TTL 和 DXL

消息队列主要是消息的“发布-订阅”，但是通过一定方式也很好地实现了”延迟“功能。

**具体如下：**

假如一条消息需要延迟 30 分钟执行，我们就设置这条消息的有效期为 30 分钟，同时为这条消息配置死信交换机和死信 routing_key，并且不为这个消息队列设置消费者，那么 30 分钟后，这条消息由于没有被消费者消费而进入死信队列，此时我们有一个消费者就在“蹲点”这个死信队列，消息一进入死信队列，就立马被消费了。

![image-20230404171047481](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230404171047481.png)



### 总结：

对于比较简单的系统，可以使用数据库轮询的方式。数据量稍大，实时性稍高一点的系统可以使用JDK延迟队列（也许需要解决程序挂了，内存中未处理任务丢失的情况）。如果需要分布式横向扩展的话推荐使用Redis的方案。但是对于系统中已有RabbitMQ，那RabbitMQ会是一个更好的方案。









# Java 继承和权限访问符号

继承不是完成得到父类的所有特性，只是拥有了父类的特性，权限访问符号控制访问父类特性的范围。

> 大多数时候父类属性定义为 private，但是可以子类访问是因为声明了public 的 set/get 方法进行获取。



![image-20230329102830792](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102830792.png)

![image-20230329102846865](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102846865.png)



# Java Mapstruct list 转换失败原因

> 转化 List<> 集合时必须有 实体转化

![image-20230329102908023](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102908023.png)

![image-20230329102922957](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102922957.png)



# Java ThreadLocal 的使用

![image-20230329103035153](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329103035153.png)



# Java 对象浅拷贝

> java.util.ConcurrentModificationException: null

![image-20230329103053520](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329103053520.png)





# Java 反射与动态代理

### 前提



### 一、反射

**获取类的字段有两种方式：**

* getFields()：获得某个类的所有的公共（public）的字段，包括父类中的字段
* getDeclaredFields()：获得某个类的所有声明的字段，即包括 public、private 和 proteced，但是不包括父类的申明字段



**获取类方法：**

* getMethods 获取所有公有方法 
* getDeclaredMethods 获取本类所有方法



**创建实例：**

* 用 Class 对象的 newInstance 方法
* 用 Constructor 对象的 newInstance 方法



### 二、静态代理

静态代理是指代理模式，通过代理对对象的进行访问和附加操作。

如下所示，Proxy 聚合了对象，所以具有对象的所有功能，可以通过 Proxy 对对象进行访问，而且还能在不改变对象的基础上，附加操作。

![image-20230616164046228](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230616164046228.png)



但是，显而易见的是，Proxy 会越来越臃肿，首先 Proxy 和其它对象的关联关系是提前确定了的，所以需要手动编写大量的 Proxy。所以我们需要一种动态代理的方式，即运行时可以动态地修改和添加代理行为。



### 三、动态代理

在 Java 中的【动态代理】是基于【反射】实现的一种技术，它可以在运行时动态地生成代理类来代理已有的类或接口，实现对目标对象的增强或拦截。

如下获取类的字段和获取类方法例子，获取使用了某个注解的类，可以往类中注入某些自定义实例。获取使用了某个注解的方法，可以对该方法实现额外的操作，而又不需要改动原生方法。如下图所示。

![image-20230213122621750](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230213122621750.png)



#### 1、类原生对象

![image-20230213122319879](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230213122319879.png)





#### 2、代理对象

> 代理是生成新的对象，而不是原生的类对象 。对象内部如果有其它的调用，内部调用的是原生的。

下面是使用 Spring 提供的动态代理方式， AOP 方式。使用反射获取 AOP 的切点后会发现，它得到的是代理对象，而不是原生对象。如下所示。

![image-20230213122057278](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230213122057278.png)



![image-20230213121958916](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230213121958916.png)



#### 3、原生对象和代理对象的关系（重要）

代理对象是原生对象的一个包装或者扩展，它们共同实现了相同的接口或者继承了相同的父类。

当我们调用代理对象时的流程是这样的：

1. 当某个对象被代理后，后续的请求会通过代理对象来处理，而不是直接经过原生对象。
2. 代理对象会持有对原生对象的引用，并在适当的时候将请求转发给原生对象处理。



![image-20230616170108004](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230616170108004.png)







# Java 责任链模式

### 前提

> [在spring中优雅的使用责任链模式 ](https://blog.csdn.net/lishuzhen5678/article/details/125657672?spm=1001.2014.3001.5502)



### 1、问题的背景

* 商城中的订单支付成功之后，有非常多的后续处理步骤，并且要求有一定的扩展性。
* 订单支付成功之后，第1步，需要更新订单状态为支付成功；第2步，向预订大表发送订单相关数据；第3步，向申请单大表发送订单相关数据；第3步，进行一些后置处理逻辑，一些资源释放工作。并且，要实现一定的扩展性，因为，订单数据，还得通过OpenFeign RPC调用，发送给其他业务线，这里一定要考虑业务的扩展性，所以选择使用责任链模式。



### 2、模式的定义与特点

在责任链模式中，客户只需要将请求发送到责任链上即可，无须关心请求的处理细节和请求的传递过程，请求会自动进行传递。所以责任链将请求的发送者和请求的处理者解耦了。



### 3、模式的结构与实现

通常情况下，可以通过数据链表来实现职责链模式的数据结构。

**职责链模式主要包含以下角色：**

* 抽象处理者（Handler）角色：定义一个处理请求的接口，包含抽象处理方法和一个后继连接。
* 具体处理者（Concrete Handler）角色：实现抽象处理者的处理方法，判断能否处理本次请求，如果可以理请求则处理，否则将该请求转给它的后继者。
* 客户类（Client）角色：创建处理链，并向链头的具体处理者对象提交请求，它不关心处理细节和请求的传递过程。



### 4、总结

责任链模式的本质是解耦请求与处理，让请求在处理链中能进行传递与被处理；理解责任链模式应当理解其模式，而不是其具体实现。责任链模式的独到之处是将其节点处理者组合成了链式结构，并允许节点自身决定是否进行请求处理或转发，相当于让请求流动起来。





# 为什么要使用 Tomcat？

首先，tomcat 是开源的，基于 java 语言开发的，部署 web 项目的容器。

普通的 html，浏览器可以直接搞定，可以不需要 tomcat 等部署，但是如 servlet 这些，浏览器没法直接将里面的内容解析出来吧。这些就需要一个工具进行处理数据，以让浏览器能够在访问主机的时候，将这些信息正确的识别出来。

可以帮我们对接http请求（做些通用处理），然后将请求转发到我们的 servlet 处理器进行处理，我们只需要把自己的业务处理放在servlet 的 service 方法即可，不需要关注其他多余的事情。

 

**具体处理的方法：**

1. 首先要使用 http 访问到你的 web 应用你服务器需要开一个端口来监听请求吧？
2. 既然使用的是 http 协议，那么需要解析来自网络的 http 请求吧？
3. 解析了之后要访问到对应的应用系统吧？
4. 系统处理了请求之后返回的结果集你需要返回给用户让用户能在浏览器中展示吧？



### 总结：

中间件就是帮你完成了这些事情而已：开启监听端口监听用户的请求，解析用户发来的 http 请求然后访问到你指定的应用系统，然后你返回的页面经过 tomcat 返回给用户。





# Java 单例模式

### 双重校验锁

~~~java
public class Singleton {

private volatile static Singleton uniqueInstance;

private Singleton() {
}

public static Singleton getUniqueInstance() {
 //先判断对象是否已经实例过，没有实例化过才进⼊加锁代码
 if (uniqueInstance == null){
     //类对象加锁
     synchronized (Singleton.class) {
         if (uniqueInstance == null){
             uniqueInstance = new Singleton();
         }
     }
 }
 return uniqueInstance;
}
~~~

【解释】

* 第一次判断是否为 null。第一次判断是在 synchronized 同步代码块外，如果已经创建了 Singleton 对象，就不用进入同步代码块，不用竞争锁，直接返回前面创建的实例即可，这样大大提升效率。
* 第二次判断是否为 null。第二次判断原因是为了保证同步。如果线程 A 通过了第一次判断，进入了同步代码块，但是还未执行，线程 B 就进来了（线程 B 获得 CPU 时间片），线程 B 也通过了第一次判断（线程 A 并未创建实例，所以 B 通过了第一次判断），准备进入同步代码块，假若这个时候不判断，就会存在这种情况：线程 B 创建了实例，此时恰好 A 也获得执行时间片，如果不加以判断，那么线程 A 也会创建一个实例，就会造成多实例的情况。





# Java getClassLoader()

**前提：**

* getClassLoader()：取得该Class对象的类装载器
* 类装载器负责从Java字符文件将字符流读入内存，并构造Class类对象。通过它可以得到一个文件的输入流



装载类的过程非常简单：查找类所在位置，并将找到的Java类的字节码装入内存，生成对应的Class对象。

**例子：**

> // 使用类加载器加载mybatis的配置文件（它也加载关联的映射文件）
>
> String resource = "conf.xml";
>
> InputStream is = Test1.class.getClassLoader().getResourceAsStream(resource);
>
> 
>
> // 构建sqlSession的工厂
>
> SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is);



![image-20230222174506081](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230222174506081.png)



# Java Yaml语法

### 字符串

1、字符串默认不使用引号表示

> str: this is string

2、如果字符串之中包含空格或特殊字符，需要放在引号中

> str: '内容： 字符串'

3、单引号之中如果还有单引号，必须连续使用两个单引号转义

> str: 'labor’‘s day'

4、字符串可以写成多行，从第二行开始，必须有一个单空格缩进，换行符会被转为空格

> str: 我是一段
>
> 多行
>
> 字符串

5、多行文本时， 用 |时，表示后面的文字每一行都被视为独立的数据(字符串) ，用>时表示只有在遇到空白行或者缩进改变时，才认为是新的数据(字符串) 

![image-20230301103812119](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230301103812119.png)

> 输出：
>
> {'name': 'Mark McGwire', 'accomplishment': 'Mark set a major league home run record in 1998.\n  home run record in 1998.\n\nhello\n', 'stats': 'Mark set a major league\nhome run record in 1998.\n  home run record in 1998.\nhello\n'}



6、使用`+`保留文字末尾的换行，`-`表示删除字符串末尾的换行

![image-20230301103956240](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230301103956240.png)

> 输出：
>
> {'name': 'Mark McGwire', 'keep': '\nMark set a major league\nhome run record in 1998.\nhello\n\n\n\n', 'strip': '\nMark set a major league home run record in 1998. hello'}



**例子：**

![image-20230301104524373](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230301104524373.png)

![image-20230301104917040](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230301104917040.png)



![image-20230301104959970](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230301104959970.png)







# Java JVM分析

### jstack

用于查看jvm线程堆栈的常用工具命令，可以获取每个线程内部的调用链以及每个线程当前的运行状态从而可以分析出死锁、循环、响应慢等性能问题



![image-20230302102656158](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230302102656158.png)



### Jstat

jstat是jdk提供的一个监控小工具，可以监控jvm的运行状态，类加载情况，jvm内存使用和GC垃圾回收以及JIT编译信息等数据。

* S0C、S1C、S0U、S1U：Survivor 0/1区容量（Capacity）和使用量（Used）
* EC、EU：Eden区容量和使用量
* OC、OU：年老代容量和使用量
* PC、PU：永久代容量和使用量
* YGC、YGT：年轻代GC次数和GC耗时
* FGC、FGCT：Full GC次数和Full GC耗时



![image-20230302102815049](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230302102815049.png)



### jstack日志分析工具

> fastthread.io







# Java 正则表达式的 **.*？**

 它表示匹配任意字符到下一个符合条件的字符

### 例子

> 正则表达式`a.*?xxx` 可以匹配 `abxxx` `axxxxx` `abbbbbxxx`



`*` 匹配0或多个正好在它之前的那个字符。例如正则表达式。*意味着能够匹配任意数量的任何字符。
`?` 匹配0或1个正好在它之前的那个字符。注意：这个元字符不是所有的软件都支持的。
`.*`是指任何字符0个或多个，
`.?`是指任何字符0个或1个.

**`.*`具有贪婪的性质，首先匹配到不能匹配为止，根据后面的正则表达式，会进行回溯。**
**`.*？`则相反，一个匹配以后，就往下进行，所以不会进行回溯，具有最小匹配的性质。**

> ？表示非贪婪模式，即为匹配最近字符 如果不加?就是贪婪模式 a.*bc 可以匹配 abcbcbc







# 注册中心 Zookeeper 、Nacos 和 Eureka

### 前提

> [eureka、zookepeer、nacos三者的关系](https://zhuanlan.zhihu.com/p/410423784)
>
> Zookeeper和Nacos都是用于服务发现、配置管理和分布式协调的开源软件。



**过半机制：**

* 消息广播：

  集群中zookeeper在数据更新的时候，通过leader节点将将消息广播给其他follower节点，采用简单的两阶段提交模式，先request->ack->commit，当超过一半的follower节点响应可以提交就更新代码。

* 崩溃恢复：

  当leader不可用时，或者超半数follower投票得出leader不可用，那么会重新选举，这段期间zookeeper服务是不可用的。通过最新的 xid来选举出新的leader，选举出来后需要将新的leader中的数据更新给超过半数的follower节点才能对外提供服务。



**C、A、P三者的定义：**

* 一致性 *Consistency* ：

  对于客户端的每次读操作，要么读到的是最新的数据，要么读取失败。换句话说，一致性是站在分布式系统的角度，对访问本系统的客户端的一种承诺：要么我给您返回一个错误，要么我给你返回绝对一致的最新数据，不难看出，其强调的是数据正确。

* 可用性 *Availability* ：

  任何客户端的请求都能得到响应数据，不会出现响应错误。换句话说，可用性是站在分布式系统的角度，对访问本系统的客户的另一种承诺：我一定会给您返回数据，不会给你返回错误，但不保证数据最新，强调的是不出错。

* 分区容忍性 *Partition tolerance*：

  由于分布式系统通过网络进行通信，网络是不可靠的。当任意数量的消息丢失或延迟到达时，系统仍会继续提供服务，不会挂掉。换句话说，分区容忍性是站在分布式系统的角度，对访问本系统的客户端的再一种承诺：我会一直运行，不管我的内部出现何种数据同步问题，强调的是不挂掉。



### web2.0数据请求模型框架

> 使用Nginx服务器进行负载均衡

但是，如果需要进行增加模块或者对一些应用进行扩展，还需要在nginx上面配置一遍，对运维人员非常不友好，而且服务太多了不好管理。



![image-20230307164944176](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230307164944176.png)



### web3.0微服务框架

> 使用服务管理组件进行可扩展性管理

随着业务的发展后面服务越来越多，服务的管理越来越复杂，如果进行服务之间的调用，可能需要将各个服务的信息写入数据库等其他地方，这种情况下服务的地址就是写死的很不容易进行扩展。所以便出现了【服务管理组件】

服务管理组件应该具有的功能：

* 服务的注册：各个服务的信息可以注册在这个组件上面
* 服务的发现：服务注册在这个组件上面之后可以被其他服务获取
* 服务的治理：服务的生命状态需要被时刻关注和管理，当服务宕机或者异常时需要进行剔除等
* 服务的负载均衡：对服务的调用采用一定的负载算法来提高服务的高可用



![image-20230307165317890](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230307165317890.png)



### Zookeeper

> Zookeeper是个【CP系统】，强一致性。
>
> 集群leader挂了会重新选举，此时暂停对外服务。Zookeeper是通过TCP的心跳判断服务是否可用。

Zookeeper的功能主要是它的【树形节点】来实现的。当有数据变化的时候或者节点过期的时候，会通过事件触发通知对应的客户端数据变化了，然后客户端再请求zookeeper获取最新数据，采用push-pull来做数据更新。服务注册和消费信息直接存储在zookeeper树形节点上，集群下采用【过半机制】保证服务节点间一致性。



### Nacos

> Nacos是个【AP/AC系统】，默认AP协议，保证其高可用。
>
> 如果对一致性的要求比较高可以切换为CP协议

Nacos的配置中心和注册中心实现的是两套代码。当有数据更新的时候，直接更新【数据库】的数据，然后将数据更新的信息异步广播给Nacos集群中所有服务节点数据变更，在由Nacos服务节点更新本地缓存，然后将通知客户端节点数据变化。



### Eureka

> Nacos是个【AP系统】，保证其高可用。





# Java ConcurrentHashMap

## 一、前提

> [ConcurrentHashMap线程不安全的场景](https://blog.csdn.net/luzhensmart/article/details/108133560)

明明用了ConcurrentHashMap可是始终线程不安全



## 二、ConcurrentHashMap 线程不安全的场景

ConcurrentHashMap的线程安全指的是，它的每个方法单独调用（即原子操作）都是线程安全的，但是代码总体的互斥性并不受控制。

> map.put(KEY, map.get(KEY) + 1);  

实际上并不是原子操作，它包含了三步：

1. map.get
2. 加1
3. map.put

首先，其中第1和第3步，单独来说都是线程安全的，由ConcurrentHashMap保证。

但是由于在上面的代码中，map本身是一个共享变量。当线程A执行map.get的时候，其它线程（线程B）可能正在执行map.put，这样一来当线程A执行到map.put的时候，线程A的值就已经是脏数据了，然后脏数据覆盖了真值，导致线程不安全。

简单地说，ConcurrentHashMap的get方法获取到的是此时的真值，但它并不保证当你调用put方法的时候，当时获取到的值仍然是真值



## 三、使用 synchronized 关键字修饰保证线程安全

例子1

![image-20230313140252776](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230313140252776.png)



例子2

![image-20230313140501181](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230313140501181.png)



## 四、总结

《Java高并发实践编程》中提到，多线程问题往往是发生在写操作时。对共享对象或变量的操作时，需要特别关注写操作时的多线程安全问题。

synchronized 关键字判断对象是否是它属于锁定的对象，本质上是通过 == 运算符来判断的。换句话说，上面的代码中，可以采用任何一个常量，或者每个线程都共享的变量，或者 MyTask 类的静态变量，来代替 map。只要该变量与 synchronized 锁定的目标变量相同（==），就可以使 synchronized 生效

如下图例子所示，lock 变量是 MyTask 类的静态变量，属于类本身，每个线程都会共享，所以 synchronized 锁的都是Mytask类。

![image-20230313140958481](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230313140958481.png)





# Spring Aop 中切点表达式 execution 的写法

### 前提

**表达式语法：**

> execution(返回值类型 包名.类名.方法名(参数))

* 返回值类型、包名、类名、方法名可以使用【*】号代替表示任意
* 包名与类名之间一个点代表当前包下的类【.】，两个点代表当前包及其子包下的类【..】
* 参数列表可以使用两个点表示任意个数的任意参数列表【(..)】

### execution的写法

1、表示修饰符为public，无返回值 aop包下target类的method方法

> execution(public void com.aop.target.method()) 



2、表述默认修饰符，无返回值，aop包下的target类下的任意参数的任意方法

> execution(void com.aop.target.*(..))



3、表示任意返回值、aop包下任意类的任意参数的任意方法（此类写法最为常用）

> execution(* com.aop.*.*(..))



4、表示任意返回值、aop包下极其子包下的任意类的任意参数的任意方法

> execution(* com.aop..*.*(..))



![image-20230315183613137](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230315183613137.png)



![image-20230315183735269](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230315183735269.png)





# Spring 事务

## 一、前提

**介绍：**

* 事务有两种方式，【局部事务】和【全局事务】。
* Spring 事务管理分为【编码式事务】和【声明式事务】。
* 声明式事务管理也有两种常用的形式，【XML形式】和【@Transactional注解形式】。



## 二、声明式事务

是基于AOP，有助于用户将操作与事务规则进行解耦。本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务。

> 默认情况下，只有来自外部的方法调用才会被AOP代理捕获，也就是，类内部方法调用本类内部的其他方法并不会引起事务行为。



## 三、Spring 事务 Synchronized 同步不生效

> [synchronized没能锁住方法](https://blog.csdn.net/jiuweihu521/article/details/115373325)

>[Spring事务中的Synchronized同步为什么不生效](https://blog.csdn.net/Rambo_Yang/article/details/119885524?spm=1001.2014.3001.5502)



![image-20230328180231725](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230328180231725.png)

上面代码在多线程下会出问题，事务范围大与锁的范围，如果锁释放后事务还没提交，就会出现“没锁住”的问题。



解决如下：同步代码块在事物之前开启，如下图所示。

![image-20230328180408287](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230328180408287.png)



![image-20230328180812679](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230328180812679.png)



# Java 关键字 Atomic

## 一、前提

> [为什么volatile不能保证原子性而Atomic可以](https://blog.csdn.net/weixin_34190136/article/details/86275092?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522167953908216800215012237%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=167953908216800215012237&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-86275092-null-null.article_score_rank_blog&utm_term=%E5%8E%9F%E5%AD%90&spm=1018.2226.3001.4450)



**符合操作：**

* 类似i++这样的"读-改-写" 复合操作，是不具有原子性

* 注意  --i、++i不是原子操作，其中包含有3个操作步骤：第一步，读取i；第二步，加1或减1；第三步：写回内存



## 二、volatile 没有原子性

> 不要将volatile用在getAndOperate场合，仅仅set或者get的场景是适合volatile的

如果让一个volatile的integer自增（i++），其实要分成3步：

1. 读取volatile变量值到local； 
2. 增加变量的值；
3. 把local的值写回，让其它的线程可见

![image-20230323104306758](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230323104306758.png)

注意中间的几步（从Load到Store）是不安全的，中间如果其他的CPU修改了值将会丢失。



## 三、AtomicXXX 具有原子性和可见性

> 核心是通过CAS（比较并交换）指令实现

Atomic 的源码里也用到了 volatile，但只是用来读取或写入。写入时会使用【CAS】，【CAS】是基于乐观锁的，也就是说当写入的时候，如果寄存器旧值已经不等于现值，说明有其他 CPU 在修改，那就继续尝试，不断循环重试【自旋】。所以这就保证了操作的原子性。

如下图所示。

![image-20230323104502846](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230323104502846.png)



![image-20230323110419650](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230323110419650.png)



应用如下图所示。

![image-20230323110151400](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230323110151400.png)



## 四、AtomicBoolean , AtomicInteger, AtomicLong, AtomicReference

这四种基本类型用来处理布尔，整数，长整数，对象四种数据。

### 1、set( )、get( )

* 可以原子地设定和获取atomic的数据。类似于volatile，保证数据会在主存中设置或读取

### 2、getAndSet( )

- 原子的将变量设定为新数据，同时返回先前的旧数据

- 其本质是get( )操作，然后做set( )操作。尽管这2个操作都是atomic，但是他们合并在一起的时候，就不是atomic。在Java的源程序的级别上，如果不依赖synchronized的机制来完成这个工作，是不可能的。只有依靠native方法才可以。

  

### 3、compareAndSet( ) 、weakCompareAndSet( )

* 这两个方法都是conditional  modifier方法。这2个方法接受2个参数，一个是期望数据(expected)，一个是新数据(new)；如果atomic里面的数据和期望数据一致，则将新数据设定给atomic的数据，返回true，表明成功；否则就不设定，并返回false。

  

### 4、incrementAndGet( )

* 对于AtomicInteger、AtomicLong还提供了一些特别的方法。

* getAndIncrement(  )、incrementAndGet( )、getAndDecrement( )、decrementAndGet ( )、addAndGet(  )、getAndAdd( ) 可以实现一些加法，减法原子操作。



![image-20230324172613225](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230324172613225.png)

![image-20230324172740717](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230324172740717.png)



## 五、总结

Atomic 听上去就像锁住了 volatile 的值，保证其它线程不可以访问。

具有了【排他性】，即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由 JVM 从等待队列中选择一个另一个线程进入。



# Spring 占位符替换工具 PropertyPlaceholderHelper

### 前提

> PropertyPlaceholderHelper 可以替换${key}、{key} 各式各样的占位符，是一个比较好用的字符串替换占位符的工具类，例如：替换短信模板信息，邮件模板信息，xml报文模板信息等。



### PropertyPlaceholderHelper

![image-20230324142254145](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230324142254145.png)



![image-20230324142439018](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230324142439018.png)





# Spring 整合 WebSocket

### 前提

> [SpringBoot项目整合WebSocket](https://www.jianshu.com/p/57fbfadacfeb)



### 使用流程

1. 导出依赖
2. 配置初始化
3. 自定义 Inteceptor
4. 自定义 Handler



#### 1、依赖

> spring-boot-starter-websocket



![image-20230324181354726](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230324181354726.png)



#### 2、配置



![image-20230324181525352](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230324181525352.png)



#### 3、自定义 Inteceptor

> 【Interceptor】拦截器用来拦截ws请求，通过实现 【HandshakeInterceptor】 接口来定义握手拦截器。



【Interceptor】 这里与下面【Handler】 的事件是不同的，拦截器这里是建立握手时的事件，分为握手前与握手后。【Handler 】的事件是在握手成功后的基础上建立 socket 的连接。所以在如果把认证放在这个步骤相对来说最节省服务器资源。



![image-20230324183230166](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230324183230166.png)



#### 4、自定义 Handler

> 【Handler】 处理器用于处理ws的消息，通过继承 【TextWebSocketHandler】 类并覆盖相应方法，可以对 websocket 的事件进行处理。



![image-20230324182548901](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230324182548901.png)



### 总结

uniiot 这里使用了 Netty 实现 WebSocketBrokerService，监听 1888 端口实现 ws 连接。即使用这样可以连接到 Broker：/localhost:1888/ws/。

uniiot 还使用引入了原生 WebSocket，监听 9091 端口实现 ws 连接。即使用这样可以连接到 WebSocket 服务：/localhost:9091/ws/sub/。

这两个 WebSocket 服务器之间的联系是，Broker 收到上行消息后，会将消息给到规则引擎进行转发，其中有一项就是让消息转发到订阅池中的所有 WebSocket连接。所有通过 /localhost:9091/ws/sub/ 连接进来的客户端都会被加入到连接池。







# Java 为什么局部变量不存在线程安全问题

### 前提

[终于弄懂了为什么局部变量是线程安全的](https://www.cnblogs.com/binghe001/p/12808419.html)

[Java 线程与进程](https://cocola6s6.github.io/java%E5%B9%B6%E5%8F%91-%E8%BF%9B%E7%A8%8B%E7%BA%BF%E7%A8%8B2/)

[Java 内存区域](https://cocola6s6.github.io/java%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F%E4%B8%8E%E5%86%85%E5%AD%98%E6%BA%A2%E5%87%BA%E5%BC%82%E5%B8%B8/)



### 局部变量存放在哪里？

局部变量的作用域在方法内部，当方法执行完，局部变量也就没用了。可以这么说，方法返回时，局部变量也就“消亡”了。此时，我们会联想到调用栈的栈帧。没错，局部变量就是存放在调用栈里的。

那全局变量呢？如果一个变量需要跨越方法的边界，就必须创建在堆/方法区里。

堆里放的是创建的对象，方法区里放的是已被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据；



### JVM 运行时数据区域

可以看到，JVM 内存区域分为了【线程共享】和【线程私有】两部分。

所以，存放在堆或方法区的东西可以线程共享，后果就是会有线程安全问题。存放在栈的东西是不会被其它线程访问到的，好处就是不会有线程安全问题。

![image-20230330103238502](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230330103238502.png)



### 栈帧图

一个线程对应一个 JVM Stack。JVM Stack 中包含一组 Stack Frame。

线程每调用一个方法就对应着 JVM Stack 中 Stack Frame 的入栈，方法执行完毕或者异常终止对应着出栈。

![image-20230330104236534](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230330104236534.png)



# Java 元空间

### 前提

[java内存区域](https://cocola6s6.github.io/java%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F%E4%B8%8E%E5%86%85%E5%AD%98%E6%BA%A2%E5%87%BA%E5%BC%82%E5%B8%B8/)

[java8之元空间](https://blog.csdn.net/weixin_38308374/article/details/110679083)



### JVM 运行时数据区域

Java里面【垃圾回收】效果最差的是永久代，而且永久代溢出也是一个非常常见的问题。所以才将永久代的数据就不断的被移到其他位置。

![image-20230403115524387](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230403115524387.png)



### 永久代和方法区

> 永久代是方法区的一种实现方式。

hotspot是JVM模型下的一种实现。方法区是java规范中定义的，只有hotspot才有永久代。hotspot命名为永久代是因为垃圾回收效果很差，大部分的数据会一直存在直到程序停止运行。

永久代里面一般存储类相关信息，比如类常量、字符串常量、方法代码、类的定义数据等，如果要回收永久代的空间，需要将类卸载，而类卸载的条件非常苛刻，所以空间一般回收很难。当程序中有大量动态生成类时，这些类信息都要存储到永久代，很容易造成方法区溢出。



### 元空间

> 元空间也是方法区的一种实现。

在java8里面，元空间替代了永久代，原来存放于永久代的类信息现在放到了元空间，我们再也不会看到PermGen error的异常了。但是尽管没有了PermGen error异常，因为类的信息存放在元空间，所以如果元空间设置的过小，而系统需要加载很多类，类没法及时被GC的话那么还是会造成元空间内存溢出。



### 类加载器

元空间是按照类加载器分配空间的，也就是说类加载器加载了一个类，元空间分配给这个类的空间其实是分配给的类加载器，不同的类加载器占用不同的空间，它们之间不共享类信息，如果程序中有大量的类加载器，而它们加载的类非常少，那么有可能会造成大量的空间浪费，而且还有可能造成java频繁GC而无法回收内存的现象。如果发现类加载器已经失效了，java可以直接将该类加载器对应的空间整体回收，不过类加载器一直存活的话，该类加载器加载的类所占据的空间不会被回收。



### 类被回收的过程

回收的过程是存在【依赖关系】。类 --> 类对象 --> 类加载器对象 --> 类实例

1. 该类的所有实例对象都已经被回收。
2. 加载该类的类加载器对象已经被回收。
3. 对应得类对象已经被回收。



### 类加载器什么时候被回收   

回收过程是存在【依赖关系】。类加载器  --> 类对象  --> 类加载器对象

1. 类加载器加载的类对象已经被回收。
2. 对应的类加载器对象已经被回收。



> 所以 通过代理生成的类不会被回收，因为它的类加载器是系统类加载器。代理类是Proxy类。



### 什么时候进行类加载

> 第一次需要使用类信息时加载。 类加载的原则：延迟加载，能不加载就不加载。



触发类加载的几种情况： 

1. 调用静态成员时
2. 第一次 new 对象的时候
3. 加载子类会先加载父类。（覆盖父类方法时所抛出的异常不能超过父类定义的范围） 



1、调用静态成员时：

* 此时会加载静态成员真正所在的类及其父类。
* 通过子类调用父类的静态成员时，只会加载父类而不会加载子类。
* 如果静态属性有 final 修饰时，则不会加载。



2、第一次 new 对象的时候

* 第二次再 new 同一个类时，不需再加载。



3、加载子类会先加载父类

* 覆盖父类方法时所抛出的异常不能超过父类定义的范围



### 依赖关系

类加载的时候，JVM会记录另外一层【类加载器间的依赖关系】，如果A类中有B类的类实例，此时【A依赖于B】，那么JVM会认为【A类加器依赖于B类加载器】。

只要这种关系建立起来，会一直持续到A类加载器被回收，B类加载器才会被回收。即使主动使用clear清除AB之间的依赖关系。



![image-20230403140030422](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230403140030422.png)







# Spring 序列化

## 一、前提

> [Java序列化](https://cocola6s6.github.io/%E5%BA%8F%E5%88%97%E5%8C%96/)

Spring默认序列化方式是什么？



## 二、Spring 的序列化方式

> 在 Controller 中使用 @ResponseBody 注解即可返回 Json 格式的数据。这些注解之所以可以进行Json与JavaBean之间的相互转换，就是因为HttpMessageConverter发挥着作用

我们的类并没有实现 Serializable 接口，实际上这是 Spring 框架帮我们做了一些事情，Spring 并不是直接把 User 对象进行网络传输，而是先把 User 对象转换成json 格式的字符串，然后再进行传输的，而 String 类实现了 Serializable 接口并且显示指定了 serialVersionUID 。



## 三、HttpMessageConverter

![image-20230420142455875](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230420142455875.png)



## 四、自定义消息转换器

> 使用 Spring 或者第三方提供的 HttpMessageConverter（如FastJson，Gson，Jackson）

存在的问题就是，如 Jackson，没有值时默认为 null 值，而不是空值，如果需要可以重写进行配置。



空值如下图所示。

![image-20230420143712025](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230420143712025.png)





null 值如下图所示。

![image-20230420143733028](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230420143733028.png)



## 五、不要莫名其妙地判空

【问题】接收前端的对象时，你是否还担心这里出现空指针呢？

不需要担心。在 Java 中，只有在尝试对 null 对象进行操作时才会抛出空指针异常。然而，在 Spring Boot 中，当使用 @RequestBody 注解接收请求 Body 中的JSON/XML 数据时，如果反序列化后的 Java 对象对应的属性为 null，调用该属性的 getter 方法不会导致空指针异常。使用 null 值才会报空指针，比如

> null.toString() // 空指针异常

如下图所示。

![image-20230424174134788](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230424174134788.png)



【问题】你是否莫名其妙地判空？

如下图所示。

YuOrderListResponseVo 对象已经创建，里面的属性默认为 null。但是这里为什么判空了，是因为要执行的枚举类的有问题。

![image-20240110175349318](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240110175349318.png)



其实根本问题在这，本身属性为 null 是正常的，只是枚举类的使用有问题。

![image-20240110175702705](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240110175702705.png)







# Spring 注解 @RequiredArgsConstructor

`@RequiredArgsConstructor` 是一个Lombok注解，它会自动生成一个包含所有被 `final` 修饰的字段的构造函数，特别注意这里的【Map<String, RepairBaseProcess> repairOrderMap】，它自动生成了key和value，其中key就是compoent的名称，value就是compoent的实例。

![image-20230519113040601](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230519113040601.png)



![image-20230519112657440](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230519112657440.png)



这样的好处是什么呢？就简化了我们自己去初始化map，对比一下原来的写法。

![image-20230519112930296](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230519112930296.png)









# Maven maven-shade-plugin

### 前提

> [maven-plugin-shade 详解 ](https://www.cnblogs.com/lkxed/p/maven-plugin-shade.html)



### maven-plugin-shade 插件的作用

* 把整个项目（包含它的依赖）都打包到一个 "uber-jar" 中
* shade - 即重命名某些依赖的包



【问题】为什么要将整个项目（包含它的依赖）都打包到一个 jar 中？

* 将依赖打包到一个JAR中的主要原因是简化应用程序的部署和依赖管理，以及减少与环境和其他应用程序的依赖冲突。



【问题】那不使用 maven-plugin-shade 时，最后也是生成一个 jar 包，这两个 jar 有什么不同么？

![image-20230605142649593](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230605142649593.png)



效果对比如下图所示。

![image-20230605143018873](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230605143018873.png)



![image-20230605143121415](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230605143121415.png)





### maven-shade-plugin 的使用

![image-20230605142322647](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230605142322647.png)





# Java 线程池

## 一、前提

[Java 线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)



## 二、ThreadPoolExecutor 的运行流程

ThreadPoolExecutor 是核心，如何同时维护线程和执行任务的呢，如下图所示。

![image-20240102181530621](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240102181530621.png)



## 三、线程和任务解耦

线程池在内部实际上构建了一个生产者消费者模型。线程池的运行主要分成两部分：任务管理、线程管理。任务管理是生产者，线程管理是消费者。



### 1、任务管理

任务管理部分充当生产者的角色，当任务提交后，【线程池】会判断该任务后续的流转

1. 直接申请线程执行该任务
2. 缓冲到队列中等待线程执行
3. 拒绝该任务。



【问题】对比使用线程池时的区别是什么？

任务只有两种情况。线程资源足够时，一个线程处理任务。线程资源不够时，抛出异常。



### 2、线程管理

线程管理部分是消费者，它们被统一维护在【线程池】内，根据任务请求进行线程的分配，当线程执行完任务后则会继续获取新的任务去执行，最终当线程获取不到任务的时候，线程就会被回收。



【问题】对比使用线程池时的区别是什么？

一个线程处理一个任务，线程资源频繁申请/销毁资源和调度。无法控制资源分布，资源完全被任务控制。







## 四、阻塞队列

如上图所示，线程池对任务得管理，主要依靠的数据结构是【阻塞队列】。



【问题】为什么使用队列而不是链表呢，链表的插入和删除的效率高。

主要是任务的调度和队列的先进先出特性匹配。其次，队列可以限制任务的数量，避免系统资源被无限制地占用。





类似的，定时器也是使用队列来实现的..............................参考【定时器和时间轮】







# Java 线程池实现原理及其在美团业务中的实践

### 一、前提

[Java 线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)



### 二、线程池解决的问题是什么

线程池解决的核心问题就是资源管理问题。如下所示。

1. 频繁申请/销毁资源和调度资源，将带来额外的消耗，可能会非常巨大。
2. 对资源无限申请缺少抑制手段，易引发系统资源耗尽的风险。
3. 系统无法合理管理内部的资源分布，会降低系统的稳定性。

为解决资源分配这个问题，线程池采用了“池化”（Pooling）思想。池化，顾名思义，是为了最大化收益并最小化风险，而将资源统一在一起管理的一种思想。



### 三、线程池在业务中的实践

#### 1、快速响应用户请求

**描述**：用户发起的实时请求，服务追求响应时间。比如说用户要查看一个商品的信息，那么我们需要将商品维度的一系列信息如商品的价格、优惠、库存、图片等等聚合起来，展示给用户。

**分析**：从用户体验角度看，这个结果响应的越快越好，如果一个页面半天都刷不出，用户可能就放弃查看这个商品了。

![image-20230610161149581](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230610161149581.png)





#### 2、快速处理批量任务

**描述**：离线的大量计算任务，需要快速执行。比如说，统计某个报表，需要计算出全国各个门店中有哪些商品有某种属性，用于后续营销策略的分析，那么我们需要查询全国所有门店中的所有商品，并且记录具有某属性的商品，然后快速生成报表。

**分析**：这种场景需要执行大量的任务，我们也会希望任务执行的越快越好。



![image-20230610161253387](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230610161253387.png)



### 四、实际问题及方案思考

线程池使用面临的核心的问题在于：**线程池的参数并不好配置**。

关于线程池配置不合理引发的故障，公司内部有较多记录，下面举一些例子：

#### 1、2018年XX页面展示接口大量调用降级

**事故描述**：XX 页面展示接口产生大量调用降级，数量级在几十到上百。

**事故原因**：该服务展示接口内部逻辑使用线程池做并行计算，由于没有预估好调用的流量，导致最大核心数设置偏小，大量抛出 RejectedExecutionException，触发接口降级条件，示意图如下：



![image-20230610161505871](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230610161505871.png)



#### 2、2018 年 XX 业务服务不可用 S2 级故障

**事故描述**：XX 业务提供的服务执行时间过长，作为上游服务整体超时，大量下游服务调用失败。

**事故原因**：该服务处理请求内部逻辑使用线程池做资源隔离，由于队列设置过长，最大线程数设置失效，导致请求数量增加时，大量任务堆积在队列中，任务执行时间过长，最终导致下游服务的大量调用超时失败。示意图如下：

![image-20230610161707181](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230610161707181.png)



业务中要使用线程池，而使用不当又会导致故障，那么我们怎样才能更好地使用线程池呢？针对这个问题，我们下面延展几个方向：

1. 能否不用线程池?
2. 追求参数设置合理性？
3. 线程池参数动态化？



#### 1、能否不用线程池？

回到最初的问题，业务使用线程池是为了获取并发性，对于获取并发性，是否可以有什么其他的方案呢替代？

![image-20230610162015152](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230610162015152.png)



综合考虑，这些新的方案都能在某种情况下提升并行任务的性能，然而本次重点解决的问题是如何更简易、更安全地获得的并发性。另外，Actor模型的应用实际上甚少，只在Scala中使用广泛，协程框架在Java中维护的也不成熟。这三者现阶段都不是足够的易用，也并不能解决业务上现阶段的问题。



#### 2、追求参数设置合理性？

调研了以上业界方案后，我们并没有得出通用的线程池计算方式。并发任务的执行情况和任务类型相关，IO 密集型和 CPU 密集型的任务运行起来的情况差异非常大，但这种占比是较难合理预估的，这导致很难有一个简单有效的通用公式帮我们直接计算出结果。



#### 3、线程池参数动态化？

我们是否可以将线程池的参数从代码中迁移到分布式配置中心上，实现线程池参数可动态配置和即时生效呢？

但是，动态线程化还需要提供如下功能：

* 动态调参：支持线程池参数动态调整、界面化操作。
* 任务监控：支持应用粒度、线程池粒度、任务粒度的 Transaction 监控。
* 负载告警：线程池队列任务积压到一定值的时候能及时告警通知。
* 操作监控



#### 总结

基于以上三个方向对比，我们可以看出参数动态化方向简单有效。





# Java 高性能队列 Disruptor

### 一、前提

> [高性能队列——Disruptor](https://tech.meituan.com/2016/11/18/disruptor.html)



### 二、Disruptor 解决的问题是什么？

为了解决内存队列的延迟问题。多线程下为了解决线程安全问题，传统的队列存在性能问题。



【问题】多线程下，任务队列会发生锁等待

线程池的任务队列是一个共享的数据结构，在【队列为空或者已满的情况下】，多线程同时访问会发生锁等待，因为获取不到资源返回嘛。



### 三、ArrayBlockingQueue 的问题

ArrayBlockingQueue 在实际使用过程中，会因为加锁和伪共享等出现严重的性能问题。



### 四、Disruptor

Disruptor 通过以下设计来解决队列速度慢的问题：

* 环形数组结构
* 元素位置定位
* 无锁设计



保证线程安全一般分成两种方式：锁和原子变量。介绍一下如何实现无锁设计。整个过程通过原子变量 CAS，保证操作的线程安全。



#### 1、一个生产者

**写数据：** 先申请写入多少个元素，如果可以写入的话 RingBuffer 会返回一个最大序列号。每个生产者自己维护这个序列号，它表示下一次写入数据的开始位置。

这个序号位运算后就是数组的下标。数组长度 2^n，通过位运算，加快定位的速度。下标采取递增的形式。不用担心 index 溢出的问题。index 是 long 类型，即使 100 万 QPS 的处理速度，也需要 30 万年才能用完。



#### 2、多个生产者

**写数据：** 因为序列号 RingBuffer 来维护的，是每个生产者都有自己维护的序列号。如果位置相同，会遇到【如何防止多个线程重复写同一个元素】的问题。

所以它的解决方法是，引入了一个与 Ring Buffer 大小相同的 available Buffer 来标记相应的位置是否已经被写入了。每次通过 CAS 来判断每次申请的空间是否已经被其他生产者占据。假如已经被占据，该函数会返回失败，While 循环重新执行，申请写入空间。

如下图所示。

1. 假设线程1需要写入 3 条数据，那么申请写入时获得 3~4 的序列号。
2. 开始写入 3，并且将 buffer 也同时标记。
3. 当其它的线程想要写入时，会判断 buffer 是否已经被标记。如果当前或者其它线程发现已经被占据了，会返回失败。
4. 成功，则继续第二步。否则重新第一步，从头开始。

![image-20231221114730839](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231221114730839.png)



**读数据：** 先申请读取多少个元素，如果某个位置的元素正在被写入，那么返回给位置作为最大读取的序号。每个消费者自己维护这个序列号。



#### 3、总结

写操作是有一个过程的，它可能包括了【申请写多少个元素，判断能写多少个元素，开始写入元素，成功写入元素】这几个步骤。这些步骤需要使得写操作没有原子性。所以需要通过增加中间体记录【过程】，多线程操作时可以通过中间体的记录判断是否存在竞争、判断写入操作是否完整。读操作也类似。



### 五、Disruptor 的等待策略

多线程情况下，无论是读还是写操作，都会遇到竞争问题，这时候可以选择中断操作，也可以选择等待一会儿。

下面是一些消费者的等待策略。

![image-20230613134319189](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230613134319189.png)



生产者的等待策略：

* 暂时只有休眠 1ns







# Java 继承的使用

![image-20230808153335565](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230808153335565.png)



![image-20230808153433269](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230808153433269.png)



执行结果如下：

![image-20230808153518679](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230808153518679.png)



可以看到，当 preStart() 执行的时候，AppRootActor、RuleChainActor、NodeActor 中的 initNodeContext 也会被调用，这是为什么？

* 因为其实他们都继承了 UntypedAbstractActor，AppRootActor-->ActorContext-->UntypedAbstractActor。不要没看到 @Override 就认为没有继承。
* 所以，他们都有 preStart()，只是它被抽象在了 ActorContext 里面而已，执行时先执行子类中的方法，找不到就会去父类找。







# Java 函数式编程

## 一、前提

函数式编程可以将一个函数的执行结果再交由另一个函数去处理。Java8 通过 【Lambda 表达式】与【方法引用】等，实现函数式编程，可将其可定义为一种简洁、可传递的匿名函数。



## 二、匿名函数

当你需要创建不会被再次使用的对象，在这种情况下，适合使用匿名内部类。

如下所示，handle 方法的参数是需要一个函数接口，正常情况下的流程如下：

1. 创建对象
2. 获取对象方法

但是，我们的参数是对象的方法，对象的名称我们不关心，所以可以声明匿名对象然后获取对象方法。

~~~java
// 函数式接口
public interface AsyncEventHandler {
    void onEvent();
}

public class Test {
    private void handle(AsyncEventHandler handler) {
        // 逻辑
    }
    
    public static void main(String[] args) {
        this.handle(
            new AsyncEventHandler() {
            	@Override
                public void onEvent() {
                	System.out.println("11111111111");
            }
    	});
}
~~~



## 三、Lambda 表达式

在匿名函数的基础上，因为 handle 的参数是一个函数式接口，所以可以变形成 Lambda 表达式。忽略对象的创建，只关注入参和方法的实现。

~~~java
// 函数式接口
public interface AsyncEventHandler {
    void onEvent();
}

public class Test {
    private void handle(AsyncEventHandler handler) {
        // 逻辑
    }
    
    public static void main(String[] args) {
        this.handle(
            () -> System.out.println("11111111111");
        );
}
~~~



## 四、方法引用

Java 8 中我们可以通过 `::` 关键字来访问类的构造方法，对象方法，静态方法。通过这个方式，可以更简洁的进行 Lambda 表达式。



## 五、实际应用

如下所示。当拥有一个方法，需要根据不用的 type 进行不用的 handle 调用时，我们会将通用的方法抽离在抽象类中，然后不用的实现对应不用的实现类。

![image-20231220144011098](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231220144011098.png)





但是，当业务不复杂的时候，我们不希望创建这么多类，但是我们又需要重写方法。那么就可以使用 Lambda 忽略类和对象，只关注方法。

如下图所示。

![image-20231220145256459](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231220145256459.png)

如下所示。

~~~java
// 函数式接口
public interface AsyncEventHandler {
    void onEvent(EventInfo eventInfo);
}


// 服务端业务类型1
public class Test {
    protected final Map<EventType, AsyncEventHandler> eventHandlers = new EnumMap<>(EventType.class);
    
    public Test() {
        eventHandlers.put(
            EventType.CLIENT_REGISTER,
            (date) -> System.out.println("11111111111");
        );
        eventHandlers.put(
            EventType.CLIENT_PROTO_COMING,
            (date) -> System.out.println("11111111111");
        );
    }
  
}

~~~



当然，如果重写的逻辑比较多，也可以抽出来放在当前类中，抽离出来的逻辑方法需要和抽象方法的签名相匹配。然后通过方法引用的方式调用，如下所示。

~~~java
// 函数式接口
public interface AsyncEventHandler {
    void onEvent(EventInfo eventInfo);
}

// 服务端业务类型2
public class Test {
    protected final Map<EventType, AsyncEventHandler> eventHandlers = new EnumMap<>(EventType.class);
    
    public Test() {
        eventHandlers.put(EventType.CLIENT_REGISTER, this::onClientRegister);
        eventHandlers.put(EventType.CLIENT_PROTO_COMING, this::onClientProtocolComing);
    }
    
    // 实现1
    public void onClientProtocolComing(EventInfo data) {
        System.out.println("11111111111");
    }

    // 实现2
    public void onClientRegister(EventInfo data) {
        System.out.println("11111111111");
    }
}

~~~





# 定时器和时间轮

## 一、前提

[知道时间轮算法吗？](https://blog.csdn.net/yessimida/article/details/107772495?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-107772495-blog-127053512.235%5Ev40%5Epc_relevant_default_base&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-107772495-blog-127053512.235%5Ev40%5Epc_relevant_default_base&utm_relevant_index=1)

kafka、netty 等高性能服务器都使用了时间轮来做延迟任务，而不是使用定时器，为什么呢？



## 二、定时器和队列

和线程池一样，定时器的主要数据结构就是【队列】，因为定时器的按照时间顺序执行的需求和队列的先进先出特性匹配。

但是，对于高性能服务器而言，任务量非常多。如果前一个任务执行时间比较长，会将后面的所有任务都往后延迟执行了。于是就有了时间轮算法。



## 三、时间轮和链表

简单的时间轮如下图所示。

1. 时间轮一直在转动
2. 从时间槽里取出对应的任务，然后执行。



如下图所示，左边解决了任务的频繁新增删除的时间复杂度差的问题，右边解决了任务阻塞的问题。

![image-20240111184705070](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240111184705070.png)



时间轮展开。如下图所示。

![image-20240202161207738](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240202161207738.png)





【问题】为什么时间轮使用的是链表而不是队列呢？

每个时间槽下都可以存放多个任务。如果使用了队列，队列的很明显的缺点就是，队列的新增和删除的时间复杂度为 O(n)，而链表的是 O(1)。对于高性能服务器而言，需要频繁地新增和删除任务。





# Spring 4.x 自动注入

在Spring4.x中增加了新的特性：如果类只提供了一个带参数的构造方法，则不需要对对其内部的属性写 @Autowired 注解，Spring 会自动为你注入属性。这个特性配合 lombok 的 @RequiredArgsConstructor 使用体验很好;



【问题】@AllArgsConstructor 和 @RequiredArgsConstructor 的区别

* @AllArgsConstructor 会为类中的所有字段生成一个构造函数
* @RequiredArgsConstructor 只会为被标记为 final 或 @NonNull 的字段生成构造函数



【问题】为什么建议使用构造函数方式注入 bean

为了保证依赖关系的不可变性。因为构造函数的签名是创建对象的唯一可能方式。一旦创建了一个bean，就不能再改变它的依赖关系。



【注意】但是，不是所有的 bean 都是需要不可变的，比如，当你需要注入的 bean 是线程池类时，线程池的线程数等是需要被调整的。

如下图所示。

![image-20240124205728877](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240124205728877.png)