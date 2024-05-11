---
title: 笔记-Hadoop大数据
categories: 好的学习笔记
tags: 大数据
---


* Hadoop 分布式计算系统生态
* Hadoop Apache Storm-使用
* Hadoop Apache Flink-使用
* Apache Storm 和 Apache Flink 的比较
* Akka 和 Apache Flink
* Actor 模型



# Hadoop 分布式计算系统生态

## 前提

> [终于有人把Hadoop大数据系统架构讲明白了](https://www.51cto.com/article/712378.html)

在大数据分析和处理领域，Hadoop 兼容体系已经成为一个非常成熟的生态圈，涵盖了很多大数据相关的基础组件，包括 Hadoop、Hbase、Hive、Spark、Flink、Storm、Presto、Impala 等。



## Hadoop 体系的分层架构

* 分布式存储层
* 分布式计算资源管理层
* 分布式并行处理框架层
* 分布式引用层

![image-20230615164957929](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230615164957929.png)





# Hadoop  Apache Storm

## 前提

> [Apache Storm 有什么作用](https://www.clouderacn.cn/products/open-source/apache-hadoop/apache-storm.html)
>
> [apache storm 基本原理及使用总结](https://www.cnblogs.com/luxiaoxun/p/11146109.html)
>
> [Apache Flink 介绍](https://aijishu.com/a/1060000000025225)



uniiot 的规则引擎，使用【Disruptor、Akka】实现的一套自定义的分布式流计算系统，用来处理大量的设备数据上报。gyyx 是使用开源的 Apache Storm。



**消息队列系统：**

> 有趣的事实是，Apache Storm 在内部使用自己的分布式消息传递系统，用于其 nimbus 和主管之间的通信。



**高吞吐、低延迟和高性能：**

* 高吞吐意味着系统能够高效地处理大量的数据或请求。
* 低延迟要求系统能够快速地响应请求并返回结果。
* 高性能是一个综合的概念，它涵盖了系统的吞吐和延迟性能。

> Apache storm、Apache Flink 等都是是一个低延迟、高性能的流式计算框架。

 

**服务间调用 RPC：**

* Storm 广泛使用 Thrift Protocol 进行内部通信和数据定义。
* Topology 是由多个组件 Spout 和 Bolt 组成的数据处理流程。这些组件可以使用 Java 或其他编程语言编写，它们通过消息传递和数据流在拓扑内部进行通信。



【问题】为什么使用 Thrift  而不直接使用 RestApi 方式呢？

![image-20230605111333521](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230605111333521.png)



【问题】Apache storm 它是怎么处理容错的？（重要）

* 对于高并发的大数据处理，一个很关键的点就是对数据的容错处理。

![image-20230605151319415](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230605151319415.png)



【问题】Apache storm 为什么是无状态的？

* Apache Storm 被认为是一种无状态的计算框架，这是由其设计和特点所决定的。如【容错性】、【实时数据流处理】等。

![image-20230605180010204](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230605180010204.png)



所谓状态，就是在流式计算过程中将中间结果数据保存在内存或者文件系统中，无须每次都基于全部的原始数据来统计结果，这种方式极大地提升了系统的性能，并降低了数据计算过程的资源消耗。但是这些【状态管理】增加了系统复杂性，而且增加了容错性的的复杂性。



**流式数据：**

* 传统数据处理方式，通常是在离线环境中对一批数据集中处理，如 Oracle、Mysql 等存储的数据。流式数据处理方式，是实时的。

![image-20230605195355746](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230605195355746.png)





## Apache Storm 资源架构

![image-20230605113116368](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230605113116368.png)

### Topology、Spouts  和 Bolts 

* Topology 是拓扑的意思，是一个抽象的概念，它有多个组件 Spouts  和 Bolts 组成。
* Spouts 是数据源组件，用于从外部数据源。
* Bolts 是数据处理组件，负责对元组进行转换、过滤、聚合等操作。
* 组件之间的通信方式采用 Thrift 实现。因为组件的实现可以是不用语言，组件间的通信需要低延迟。



【问题】Storm中，Bolt 的排布的有序的吗？

![image-20230605113523251](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230605113523251.png)



### Stream 和 Tuple 

*  Stream 表示无限序列元组
*  Tuple 是 Storm 中的主要数据结构。它是有序元素的列表。默认情况下，Tuple 支持所有数据类型。



### 分组操作

* 流分组控制 Tuple 在拓扑中的【路由和传递方式】。相同组的数据被分发到同一个下游 Bolt 实例上。
* 数据在不同的 Bolt  可以进行不同的处理，在 Bolt 中可以对数据进行更【更细粒度】的划分，然后对划分处理的数据进行并行处理。



【问题】Apache Flink 的分区和 Apache Storm 的分组怎么有些类似呢？

* 功能类似，但是它们的目的是不同的。如下所示。

![image-20230606104207428](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230606104207428.png)



### 总结

Apache Storm 资源架构有利于它的内存管理。

1. 首先将数据分割为小的 Tuple 并逐个处理，这种分批处理的方式有助于降低内存使用量，防止直击内存导致内存溢出。
2. 其次 Spouts、Bolts、Grouping 操作等，更细粒度可以提高流数据处理效率。



## Apache Storm 工作流程

![image-20230605144403158](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230605144403158.png)



![image-20230605115449404](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230605115449404.png)

### Nimbus

> 也就是 Master

- 上传计算以便执行
- 在整个集群中分发代码
- 在集群中启动 worker
- 监控计算并根据需要重新分配 worker



### Supervisor

> 也就是 Worker

* 通过 Supervisor 的协调和管理。
* 负责执行和管理拓扑的任务，监控任务状态，处理任务分配。
* 管理工作节点资源，并处理故障恢复。



### ZooKeeper

协调 Storm 集群。它没有内置的类似的 Scheduler 进行资源调度。

* ZooKeeper 在 Storm 中主要用于 Nimbus 的选举和集群状态的管理。
* Zookeeper是个【CP系统】，强一致性。
* 集群 leader 挂了会重新选举，此时暂停对外服务。Zookeeper 是通过 TCP 的心跳判断服务是否可用。



### 任务和进程

* 进程是指运行在集群中的物理或虚拟机上的 Storm 实例。进程由 【Storm 集群管理器（Nimbus）】启动和监控，并与其他进程进行通信以协调任务的分配和数据的传输。
* 任务可以是 Spout 任务或 Bolt 任务，每个进程可以执行一个或多个任务





# Hadoop Apache Storm-使用

## 使用流程

1. 启动 Topology，配置 Spout 和 Bolt
2. Spout 业务
3. Bolt 业务



### 1、启动 Topology，配置 Spout 和 Bolt

1. KafkaSpout 接收消息：boxRuntimeSpout 和 boxConfigSpout 是两个 KafkaSpout，它们从 Kafka 中读取数据。boxRuntimeSpout 读取运行时数据，boxConfigSpout 读取配置数据。
2. 解析消息：接收到的消息被发送到 boxRuntimeParseBolt 和 boxConfigParseBolt 进行解析。boxRuntimeParseBolt 解析运行时数据，boxConfigParseBolt 解析配置数据。



1.  运行时数据处理：解析后的运行时数据被发送到 boxRuntimeToDBBolt 和 boxRuntimeFilterBolt。boxRuntimeToDBBolt 可能将数据存储到数据库，boxRuntimeFilterBolt 可能对数据进行过滤。
2.  过滤后的运行时数据处理：过滤后的数据被发送到多个Bolt进行进一步处理，包括 BoxToDbBolt, BatteryToDbBolt, BoxQuantityDayToDbBolt, BoxBatterySocChangeToDbBolt, BoxDoorEnableToDbBolt, BatteryCabinetToDbBolt, BoxDoorTemperatureToDbBolt, BoxDoorTemperatureAbnormalToDbBolt, BoxTemperatureToDbBolt。这些 Bolt 可能将数据存储到数据库。



1. 配置数据处理：解析后的配置数据被发送到 boxConfigToDBBolt，可能将数据存储到数据库。
2. 提交拓扑：最后，根据命令行参数，拓扑被提交到 Storm 集群运行，或者在本地运行。



如下图所示。

![image-20231221104320491](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231221104320491.png)



【问题】fieldsGrouping 使用怎么理解？

> fieldsGrouping("boxRuntimeFilterBolt", new Fields("devId"))：

它表示 BoxToDbBolt 将接收 boxRuntimeFilterBolt 发射的数据，并且数据会根据"devId"字段的值进行分组，即具有相同"devId"字段值的数据会被发送到同一个BoxToDbBolt实例。



### 2、Spout 业务

略.............



### 3、Bolt 业务

![image-20230605152401430](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230605152401430.png)



两个重要的方法：

* 【excute】方法，每次 Bolt 从流接收一个订阅的 tuple，都会调用这个方法。

* 【declareOutputFields】 方法的调用通常是在组件的初始化阶段进行的，而不是在每个元组的处理过程中。它只需要被调用一次。用来指定数据的下一个流向。





# Hadoop Apache Flink

## 前提

[Apache Flink 介绍](https://aijishu.com/a/1060000000025225)

[Flink 介绍和基本概念](https://www.jianshu.com/p/2ee7134d7373)



## Apache Flink 资源架构

![image-20230606111121472](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230606111121472.png)



![image-20230606113239893](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230606113239893.png)



### Source

* Source 是数据流处理的起点



### Stream

* 表示一系列有序的事件或记录，可以是实时生成的数据流或批量读取的数据集



### Transformation

* 对数据流进行操作和转换的操作符，用于对数据进行处理、过滤、聚合、连接等操作
* 它对一个或多个输入 Stream 进行计算处理，输出一个或多个结果 Stream



### Sink

* Sink 是数据流处理的终点



### 分区操作

* 一个 Stream 可以被分成多个 Stream 分区，一个 Operator 可以被分成多个Operator Subtask。
* 与 Apache storm 的分组功能类似，但是它们的目的是不同的。



分区操作之后，Operator 之间的 Stream 的两种模式：

* One-to-one 模式：它是有序的
* Redistribution模式：它是无序的



### 窗口、时间和状态（重要）

略....................可以看 [Flink介绍和基本概念](https://www.jianshu.com/p/2ee7134d7373) 的解释。

状态管理使得 Flink 具有更强大的容错性，它使用了可重播日志和检查点机制来实现精确一次性处理保证。而 Storm 是采取重传的机制，需要开发者自行处理幂等性。



## Apache Flink 工作流程

![image-20230606114033701](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230606114033701.png)



流程如下：

1. Flink 的 Driver 程序会将代码逻辑构建成一个 Program Dataflow，如区分source，operator，sink。
2. 再通过 Graph Builder 构建 DAG 的 Dataflow graph，构建 job，划分出 task 和 subtask。
3. Client 将 job 提交到 JobManager。
4. JobMangaer 按照 Dataflow 的 Task 和 Subtask 的划分，将任务调度分配到各个 TaskManager 中进行执行。
5. TaskManager 会将内存抽象成多个 TaskSlot，用于执行 Task 任务。



### JobManagers

> 也就是 Master

* 用来调度任务，协调 checkpoints，协调错误恢复等。
* 至少需要一个 JobManager，高可用的系统会有多个，一个 leader，其他是 standby。



### TaskManagers

> 也就是 Worker

* 用来执行数据流任务或者子任务，缓存和交互数据流。



### Client

*  Client 不是运行是和程序执行的一部分，它是用来准备和提交数据流到 JobManagers。之后，可以断开连接或者保持连接以获取任务的状态信息。

> 这里的数据流指的是：Flink 的 Driver 程序会将代码逻辑构建成一个Program Dataflow，即 source，operator，sink。再通过Graph Builder 构建 DAG 的 Dataflow graph。



### ActorSystem 

* Client 通过 Actor System 和 JobManager 进行消息通讯，接收 JobManager 返回的状态更新和任务执行统计结果。
* JobManagers 与 TaskManagers 之间的任务管理，Checkpoints 的触发，任务状态，心跳等等消息处理都是通过 ActorSystem。



# Hadoop Apache Flink-使用

## 使用流程

1. 启动 Flink，配置 DataStream、keyedStream、sink
2. sink 业务



### 1、启动 Flink，配置 DataStream、keyedStream、sink

1. 从 Kafka 中读取数据：这个应用使用 Flink 的 addSource 方法从 Kafka 中读取数据流。这些数据可能是 IoT 设备发送的实时数据。
2. 数据转换：读取的数据被转换为 BikeLocationBean 对象，这可能是表示自行车位置的数据模型。
3. 数据分组：使用 keyBy 方法按 BikeLocationBean 的 getTerminalId 方法进行分组，这可能是按设备 ID 对数据进行分组。
4. 数据输出：处理后的数据被发送到不同的 Sink 中，包括数据库和 Redis。这可能是为了将处理后的数据存储起来，以便进一步的分析和使用。

![image-20231221101818951](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231221101818951.png)



### 2、sink 业务

![image-20231221101903203](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231221101903203.png)



# Apache Storm 和 Apache Flink 的比较

## 一些问题

【问题】apache flink 和 apache storm 的比较？

* 数据流模型（重要）
* 容错性处理（重要）
* 语义
* 批处理支持
* 集群管理
* 社区和生态

![image-20230606150104174](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230606150104174.png)



#### 1、数据流模型

* flink 关注批量处理和实时流处理
* storm 关注实时流处理



#### 2、容错性处理

Apache Storm 通过使用【消息确认机制】。但在任务故障时，需要手动处理消息重复和消息丢失的情况。

看到【消息确认机制】是不是很熟悉，消息队列 RabbitMq 的容错处理也是采用消息确认机制，如果没得到确认就将消息重传。



Apache Flink 通过使用【检查点机制】【保存点机制】【故障恢复机制】。

意思就是，Flink 是有状态的，这些状态会并行保存在持久化存储系统中，如果发生故障，可以自动从存储中拿出来重新执行，也可以手动拿出来。自动检查就是检查点机制，手动检查就是保存点机制。





# Akka 和 Apache Flink

## 前提

* Apache Flink 的 RPC 实现：是基于 Scala 的网络编程库 Akka 来的。
* Akka 是一个基于 Actor 模型的并发编程框架。



## Akka 的实践

参考 [AKKA 实现的 IOT 规则引擎方案](https://github.com/Cocola6s6/iot_akka)



### 1、akka 配置

配置如下。

~~~bash
akka {
  # akka 引起的致命原因不关闭JVM
  jvm-exit-on-fatal-error = off
  log-config-on-start = on
  loglevel = "INFO"
  loggers = ["akka.event.slf4j.Slf4jLogger"]
}

# 规则处理
engine-dispatcher {
  type = Dispatcher
  executor = "fork-join-executor"
  fork-join-executor {
      # 最小并发线程数
      parallelism-min = 5
      # 最大并发线程数
      parallelism-max = 15
      # 并发因子，该值会影响线程池的最大可用线程数，具体的计算方式是： 最大线程数＝处理器个数＊parallelism-factor ,比如当服务器C PU 处理器个数是4 ， 并发因子是3 ，那么最大线程数就是1 2
      parallelism-factor = 0.25
  }
  # 对于一个Actor ，某个线程在（ 返回到线程池） 处理下一个Actor 之前能处理的最大消息数，假如设置为1，表示尽可能公平的处理消息
  throughput = 10
}

engine-mailbox
{
  mailbox-type = "akka.dispatch.NonBlockingBoundedMailbox"
  mailbox-capacity = 25000
}

# 执行节点
node-dispatcher {
  type = PinnedDispatcher
  executor = "thread-pool-executor"
  fork-join-executor {
      # 最小并发线程数
      parallelism-min = 2
      # 最大并发线程数
      parallelism-max = 12
      # 并发因子，该值会影响线程池的最大可用线程数，具体的计算方式是： 最大线程数＝处理器个数＊parallelism-factor ,比如当服务器C PU 处理器个数是4 ， 并发因子是3 ，那么最大线程数就是1 2
      parallelism-factor = 1.0
  }
  # How long time the dispatcher will wait for new actors until it shuts down
  shutdown-timeout = 1s

  # 对于一个Actor ，某个线程在（ 返回到线程池） 处理下一个Actor 之前能处理的最大消息数，假如设置为1，表示尽可能公平的处理消息
  throughput = 5
}
~~~





# Actor 模型

### 前提

> [分布式高并发下Actor模型如此优秀](https://cloud.tencent.com/developer/news/698662)



当服务使用分布式时，服务需要提高并发和并行处理，才能提高系统吞吐量。在高并发环境下，需要考虑其带来的并发安全问题。所以需要通过并发编程来解决这些问题。

【问题】并发编程时需要注意什么？

* 在并发编程中，需要仔细考虑线程安全性、同步机制、死锁和活锁、上下文切换和线程调度、锁粒度和性能、并发编程工具和库、错误处理和异常处理，以及性能调优等方面。



【问题】并发编程需要注意的问题，已经有了很多的实践方式。并发编程都有哪些方式呢？

* 我们会联想到 Java 中的锁、原子操作和线程安全的数据结构来实现线程安全等。这个方式属于【共享数据】的方式。
* 如下图所示。

![image-20230606211004572](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230606211004572.png)



### 消息传递和 Actor 模型

一般来说有两种策略用来在并发线程中进行通信：【共享数据】和【消息传递】。使用共享数据方式的并发编程面临的最大的一个问题就是数据条件竞争。处理各种锁的问题是让人十分头痛的一件事。

Actor 是一个【消息传递】的方式的并发编程模型。

* 每个 Actor 在同一时间处理最多一个消息，可以发送消息给其他 Actor，保证了单独写原则。从而巧妙避免了多线程写争夺。
* 和共享数据方式相比，消息传递机制最大的优点就是不会产生数据竞争状态。



![image-20230606212137364](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230606212137364.png)



看上图所示，Actor 模型有以下特点：

* 更加面向对象
* 无锁
* 异步
* 隔离
* 天生分布式
* 生命周期
* 容错



### 无锁

* 在使用 Java/C# 等语言进行并发编程时需要特别的关注锁和内存原子性等一系列线程问题，而 Actor 模型内部的状态由它自己维护，即它内部数据只能由它自己修改。所以使用 Actor 模型进行并发编程可以很好地避免这些问题。
* Actor 内部是以单线程的模式来执行的，类似于 redis，所以 Actor 完全可以实现分布式锁类似的应用。



【问题】如果消息太多的情况下，邮箱是否会像线程池的队列一样，需要锁等待呢？

* 线程池的任务队列是一个共享的数据结构，在【队列为空或者已满的情况下】，多线程同时访问会发生锁等待，因为获取不到资源返回嘛。
* 但是，Actor 是单线程！！！单线程啊，怎么会有竞争呢？很明显不会邮箱不会存在这个问题。



如下，engine-mailbox 的配置。

* 邮箱类型。选择有界的、非阻塞的邮箱，它在邮箱满时不会阻塞发送者，而是会抛出异常。
* 邮箱的容量。设置最多可以存储 25000 条消息。如果超过这个数量，新的消息将无法被添加到邮箱中。

~~~bash
engine-mailbox
{
  mailbox-type = "akka.dispatch.NonBlockingBoundedMailbox"
  mailbox-capacity = 25000
}
~~~





### 隔离

每个 Actor 的实例都维护这自己的状态，与其他 Actor 实例处于物理隔离状态，并非像【多线程+锁】模式那样基于共享数据。Actor 通过消息的模式与其他 Actor 进行通信，而且 Actor 之间消息的传递是真正物理上的消息传递。



【问题】Actor 之间消息的传递是真正物理上的消息传递怎么理解？

![image-20230606213303986](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230606213303986.png)



### 天生分布式

* 每个 Actor 实例的位置透明，无论 Actor 地址是在本地还是在远程机器上对于代码来说都是一样的。对于调用者来说，调用的 Actor 的位置就在本地，当然这也得益于 Actor 系统强大的路由系统。 
* 每个 Actor 的实例非常小，最多几百字节，所以单机几十万的 Actor 的实例很轻松。



【问题】对于调用者来说，调用的 Actor 的位置就在本地，当然这也得益于 Actor 系统强大的路由系统？

* 每个Actor都有一个唯一的地址标识，类似于一个本地对象的引用。

![image-20230608103005927](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230608103005927.png)



【问题】每个 Actor 都有一个唯一的地址标识，这个怎么理解，其它 Actor 怎么知道这个地呢？

* 每个 Actor 都有一个唯一的地址标识，其他 Actor 可以通过管理组件、服务发现机制或其他方式获取目标Actor的地址，并向该地址发送消息。

![image-20230608103115942](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230608103115942.png)



【问题】服务发现机制是 Actor 自带的吗？还是需要引入第三方如 Zookeeper？

![image-20230608103237214](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230608103237214.png)



### 总结

所以，既然 Actor 对于位置是透明的，任何 Actor 对于其他 Actor 就好像在本地一样。基于这个特性我们可以做很多事情了，以前传统的分布式系统，A 服务器如果想和 B 服务器通信，要么 RPC 的调用（http 调用不太常用），要么通过 MQ 系统。

但是在 Actor 系统中，服务器之间的通信都变的很优雅了，虽然本质上也属于 RPC 调用，但是对于编码者来说就好像在调用本地函数一样。其实现在比较时兴的是 Streaming 方式。





