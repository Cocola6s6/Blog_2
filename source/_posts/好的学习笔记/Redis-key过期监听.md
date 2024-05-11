---
title: Redis的key过期监听不可靠吗？
categories: 好的学习笔记
tags: Redis
---



# Rediskey 过期监听

## 一、前提

遇到了 Redis key 过期监听时间不准时的问题，key 已经失效了，但是事件晚了 6s 才到达。

[请勿过度依赖Redis的过期监听](https://juejin.cn/post/6844904158227595271?spm=a2c6h.12873639.article-detail.10.2a9c1b9aU1zxXK)



【原因】

![image-20240321112145040](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240321112145040.png)

这里面涉及了 Redis 对过期 key 的删除策略。

 

## 二、Rediskey 的过期策略

[redis-使用的过期删除策略是什么](https://www.xiaolincoding.com/redis/base/redis_interview.html#redis-%E4%BD%BF%E7%94%A8%E7%9A%84%E8%BF%87%E6%9C%9F%E5%88%A0%E9%99%A4%E7%AD%96%E7%95%A5%E6%98%AF%E4%BB%80%E4%B9%88)

* 惰性删除
* 定期删除



### 1、惰性删除

不主动删除过期的 key，当进行访问的时候，如果 key 过期了才进行删除。这样子的话，就不用通过“轮询”的方式去删除，节省了 CPU。缺点也明显，如果某个过期 key 一直不被访问的话，它所占用的内存就不会释放，造成了一定的内存空间浪费。所以，惰性删除策略对内存不友好。



所以，就需要定期删除辅助。

### 2、定期删除

每隔一段时间随机从数据库中取出一定数量的 key 进行检查，并删除其中的过期key。

注意，为了防止过度消耗 CPU，它对执行周期、执行时长、key 数量都有限制。执行时长是 20ms、key 数量是 20 个且不超过 20%。





## 三、Redis 持久化时，对过期键会如何处理的？

[Redis 持久化时，对过期键会如何处理的？](https://www.xiaolincoding.com/redis/base/redis_interview.html#redis-%E4%BD%BF%E7%94%A8%E7%9A%84%E8%BF%87%E6%9C%9F%E5%88%A0%E9%99%A4%E7%AD%96%E7%95%A5%E6%98%AF%E4%BB%80%E4%B9%88)

### 1、RDB

在 RDB 文件生成时，会对内存中的 key 进行过期检测，所以过期的 key 是不会保存到 RDB 文件中的。

在加载 RDB 文件到内存中时也一样，会进行过期检测。但是，对于从服务器来说，因为是从主服务器进行数据同步，所以它加载到内存时可以不用进行过期检测。



### 2、AOF

AOF 只有在【重写阶段】才会进行过期检测，因为重写就是为了压缩文件，肯定过滤掉过期的。



【问题】主从模式中，对过期键会如何处理？

* 从服务器不会进行过期扫描，它是被定的，它只会和主服务器进行同步。所以即使从库中的 key 过期了，如果有客户端访问从库时，依然可以得到 key 对应的值，像未过期的键值对一样返回。





