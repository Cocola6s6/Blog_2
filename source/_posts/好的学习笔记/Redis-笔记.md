---
title: 笔记-Redis
categories: 好的学习笔记
tags: Redis
banner_img: https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240108233805338.png
---



# Redis 高并发

## 一、前提

[Redis 为什么这么快](https://juejin.cn/post/7170712250880606221)



## 二、Redis 为什么用单线程

redis 核心就是，如果我的数据全都在内存里，我单线程的去操作就是效率最高的。因为内存的操作时快速的，redis 不希望这个快速读取的过程被 CPU 的上下文切换浪费时间。单线程下对绑定的内存进行操作，那么导致 redis 慢的因素就是网络带宽等外部因素了。



## 三、跳表

[Redis 为什么使用跳表而不是 B+ 树](https://juejin.cn/post/7149101822756519949)

[Redis内部数据结构之跳表](https://zhuanlan.zhihu.com/p/56941754)

对于保存在内存的数据，redis 采用了一个高效的数据结构，【跳表】。通过构建多级索引来提高查询的效率，使用了空间换时间的思路。 



### 1、什么是跳表

如下图所示。每两个结点提取一个结点到上一级，我们把抽出来的那 一级叫作索引或索引层，一直到顶层只有一个。这样子，查找过程就非常类似于一个二分查找。

单链表的查询的时间复杂度是 O(n)，跳表的查询的时间复杂度是 O(logn)。

![image-20240104143451472](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240104143451472.png)



![image-20240104143525438](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240104143525438.png)





### 2、跳表数据结构

~~~c
#define ZSKIPLIST_MAXLEVEL 32 //最大层数
#define ZSKIPLIST_P 0.25 //P


typedef struct zskiplistNode {
    robj *obj;
    double score;
    struct zskiplistNode *backward; //后向指针
    struct zskiplistLevel {
        struct zskiplistNode *forward; //每一层中的前向指针
        unsigned int span; //x.level[i].span 表示节点x在第i层到其下一个节点需跳过的节点数。注：两个相邻节点span为1
    } level[];
} zskiplistNode;


typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length; //节点总数
    int level; //总层数
} zskiplis
~~~



zskiplist 的结构很明显，分别表示整个跳表的总节点数、总层数和头尾节点。

zskiplistNode 主要需要关注以下两个属性：

* 【score】 是分值，当你将一个元素添加到有序集合时，你需要为该元素指定一个分值，用于对有序集合中的元素进行排序

*  【level】是个数组，【level】包含了指向 next 节点的指针和到达 next 节点需要需要跳过的节点数【跨度】



如下图所示。

![image-20240105144912389](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240105144912389.png)



### 3、跳表的查询

如下图所示。

![image-20240105150643371](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240105150643371.png)



如下，查询【节点23】的过程，可以看到，如果是单链表的话，时间复杂度是 6。现在使用跳表的时间复杂度是 4。

1. Level4 开始，先找到【节点7】，经过判断，7 小于目标节点，需要继续判断 next 节点。【节点7】的 next 节点为 null，到此 Level4 判断结束。
2. Level3 开始，【节点7】的 next 节点是【节点37】，经过判断，37 大于目标节点。到此 Level3 判断结束。
3. Level2 开始，【节点7】的 next 节点是【节点19】，经过判断，19 小于目标节点需要继续判断 next 节点。【节点19】的 next 节点为 null，到此 Level2 判断结束。
4. Level1 开始，【节点19】的 next 节点是【节点22】，经过判断，22 小于目标节点，需要继续判断 next 节点。

【注意】真实的比较的不仅仅是对象值，而是同时比较【score+obj】



~~~bash
##过程伪代码如下
fn find() {
	// 1.从顶层开始
	// 2.如果小于目标值,则找到next继续比较
	// 3.如果大于目标值或者为null,则到下一层从头开始比较
}
~~~





### 4、跳表的插入

[Redis源码解析-基础数据](https://juejin.cn/post/6844904004498128903)

~~~c
/*
* 计算当前插入元素层高的随机函数
*/
int zslRandomLevel(void) {
    int level = 1;
    // (random()&0xFFFF) < (ZSKIPLIST_P * 0xFFFF) 概率为1/4
    while ((random()&0xFFFF) < (ZSKIPLIST_P * 0xFFFF))
        level += 1;
    return (level<ZSKIPLIST_MAXLEVEL) ? level : ZSKIPLIST_MAXLEVEL;
}

/*
* 插入节点
*/
zskiplistNode *zslInsert(zskiplist *zsl, double score, sds ele) {
    // update存放需要更新的节点
    zskiplistNode *update[ZSKIPLIST_MAXLEVEL], *x;
    unsigned int rank[ZSKIPLIST_MAXLEVEL];
    int i, level;

    serverAssert(!isnan(score));
    x = zsl->header;
    
    
    // 第一步，收集需要更新的节点与步长信息
    for (i = zsl->level-1; i >= 0; i--) {
        /* store rank that is crossed to reach the insert position */
        rank[i] = i == (zsl->level-1) ? 0 : rank[i+1];
        // score可以重复，重复时使用ele大小进行排序
        while (x->level[i].forward &&
                (x->level[i].forward->score < score ||
                    (x->level[i].forward->score == score &&
                    sdscmp(x->level[i].forward->ele,ele) < 0)))
        {
            rank[i] += x->level[i].span;
            x = x->level[i].forward;
        }
        update[i] = x;
    }
    
    
    // 第二步， 获取随机层高，补全需要更新的节点
    level = zslRandomLevel();
    if (level > zsl->level) {
        for (i = zsl->level; i < level; i++) {
            rank[i] = 0;
            update[i] = zsl->header;
            update[i]->level[i].span = zsl->length;
        }
        zsl->level = level;
    }
    
    
    // 第三步，创建并分层插入节点，同时更新同层前一节点步长信息
    x = zslCreateNode(level,score,ele);
    for (i = 0; i < level; i++) {
        x->level[i].forward = update[i]->level[i].forward;
        update[i]->level[i].forward = x;

        /* update span covered by update[i] as x is inserted here */
        x->level[i].span = update[i]->level[i].span - (rank[0] - rank[i]);
        update[i]->level[i].span = (rank[0] - rank[i]) + 1;
    }
    
    
    // 第四步，更新新增节点未涉及层节点的步长信息，以及跳表相关信息
    /* increment span for untouched levels */
    for (i = level; i < zsl->level; i++) {
        update[i]->level[i].span++;
    }

    x->backward = (update[0] == zsl->header) ? NULL : update[0];
    if (x->level[0].forward)
        x->level[0].forward->backward = x;
    else
        zsl->tail = x;
    zsl->length++;
    return x;
}

~~~



#### 1）图解

假设现在我需要插入元素80，且获取到随机的层高为5(为了所有情况都覆盖到)。

![image-20240106141233545](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240106141233545.png)



【第一步】收集需要更新的节点与步长信息。

从最高层开始，查找插入节点的位置时，将受影响的节点和步长保存起来

* update[] 数组保存节点（红色框出来的）。
* rank[] 数组保存每层头节点与会受影响的节点中间的步长。(rank为[3, 3, 5, 6] 分别表示头节点到 i+1 层节点的步长)

![image-20240106141617725](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240106141617725.png)



【第二步】获取随机层高，补全需要更新的节点，同时可能更新跳表高度

如下虚线位置，插入层高为 5 的【节点80】

* 随机获得层高为 5。
* 因为高于当前层高，所以表层高要变，表头结点也要变。



![image-20240106142730066](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240106142730066.png)



【第三步】创建并分层插入节点，同时更新同层前一节点步长信息

如下图所示，开始执行实际插入。

* 需要调整现有层（L1~L4）受影响节点的 next 节点为【节点80】。
* 需要调整现有层（L1~L4）受影响节点的步长。如图都往前到了新的位置。



![image-20240106144522141](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240106144522141.png)



【第四步】更新新增节点未涉及层节点的步长信息，以及跳表相关信息与节点自身的相关信息

前面的步骤修改的都是【受影响节点】，即插入节点前，第四步处理【未受影响节点】

- 如果当前节点的层高比跳表高度低，那么高于当前节点层高的那些层中排在当前节点之后的节点步长信息都需要+1(因为在它和它的前一个节点之间插入了新元素)。
- 更新跳表长度与当前节点与第一层下一节点的后退指针(后退指针可以理解为只有底层链表有)。





【问题】如何进行链表的插入？

* 先断开，后插入，即把先后节点抢了，让前节点空出来。

~~~c
// 需要在update和next之间插入x

// 1.先断开update和next之间的指针,next现在被x连着了
x->level[i].forward = update[i]->level[i].forward;

// 2.update被断开,处于空闲,可以插入
update[i]->level[i].forward = x;
~~~



### 5、跳表的删除



## 四、I/O 多路复用模型

[深入学习IO多路复用 select/poll/epoll 实现原理](https://cloud.tencent.com/developer/article/2188691)



### 1、阻塞 IO

客户端代码如下所示。

~~~c
int main()
{
     int fd = socket();      // 创建一个网络通信的socket结构体
     connect(fd, ...);       // 通过三次握手跟服务器建立TCP连接
     send(fd, ...);          // 写入数据到TCP连接
     close(fd);              // 关闭TCP连接
}
~~~



服务端代码如下所示。

~~~c
int main()
{
     fd = socket(...);        // 创建一个网络通信的socket结构体
     bind(fd, ...);           // 绑定通信端口
     listen(fd, 128);         // 监听通信端口，判断TCP连接是否可以建立
     while(1) {
         connfd = accept(fd, ...);              // 阻塞建立连接
         int n = recv(connfd, buf, ...);        // 阻塞读数据
         doSomeThing(buf);                      // 利用读到的数据做些什么
         close(connfd);                         // 关闭连接，循环等待下一个连接
    }
}
~~~



如下图所示。

![image-20240108174641698](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240108174641698.png)



简单过程如下：

1. 创建 socket 后，如果网卡还没有数据给到，那么就让出 CPU，进程等待。
2. 数据到达网关后，数据从 buffer 拷贝到 socket。
3. socket 有数据了，数据从socket 从内核态拷贝到用户态，给你 CPU，进程唤醒了处理用户态数据。

总结：一次数据到达经历了两次处理器状态转换，两次阻塞。一次阻塞是 recv 操作等待套接字数据，一次是内核态数据拷贝到用户态操作。



如下图所示。

![image-20240108233805338](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240108233805338.png)



【注意】

* recv 底层的系统调用是 recvfrom。recvfrom 是一个系统调用，用于从套接字接收数据。
* 内核态数据拷贝到用户态操作的系统调用是 copy_to_user。



【注意】

* 进程调度和切换是操作系统内核程序，所以进程切换（进程上下文切换）必定在内核态发生。
* 【系统调用、中断和异常】是激活操作系统仅有的方法，此时会发生处理器状态转换，从用户态转换为内核态或者从内核态转换为用户态。



【注意】进程上下文切换和处理器状态切换

* 处理器状态转换不一定引起上下文切换，比如 IO 操作时，线程先是完成了用户态的操作，然后进入到内核态完成内核下才有权限的操作，这个过程完成了处理器状态转换，但是都是同一个线程，只是执行的任务不一样（先执行用户态任务，后执行内核态任务）。





### 2、非阻塞 IO

就是在阻塞 IO 的基础上，将等待 socket 有数据这一步变成非阻塞了，没有时 recvfrom 直接返回。

如下图所示。

![image-20240109000938804](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240109000938804.png)





上面阻塞 IO 和非阻塞 IO 都是一个进程对应一个连接，如果想要压榨单线程处理多个连接，就需要通过【select、poll、epoll】实现。如下介绍。



### 3、select

数据结构如下所示。

~~~c
int select(
    int nfds,
    fd_set *readfds, // 文件描述符集合，引用类型的参数
    fd_set *writefds,
    fd_set *exceptfds,
    struct timeval *timeout
);
~~~



使用【文件描述符集合fdsr 】保存当前进程的所有 socket 连接，这样子当网卡有数据到达时，进程就可以去 fdsr 中找到对应的 socket 进行数据处理。

如下图所示。

![image-20240109200327848](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240109200327848.png)





但是缺点也很明显，大概有如下：

* 增加了文件描述符集合维护所有 socket 连接，同时改文件也需要在用户态和内核态之间拷贝两次。
* 因为不知道具体哪个 socket，而且返回了所有，所以进程需要循环所有文件描述符集合。
* 文件描述符集合大小有限。



【问题】什么是文件描述符 fd

Linux 中一切皆文件。进程使用文件描述符来标识一个打开的文件。通过 socket 通信，实际上就是通过文件描述符 fd 读写文件。socket 与 fd 是一一对应的。



### 4、poll

poll 解决了 select 的数量大小问题，使用复合数据结构以链表的形式存储。

数据结构如下所示。

~~~c
struct pollfd {
    int fd; // 文件描述符
    short events;
    short revents;
};
~~~





### 5、epoll

对比前面的 select 和 poll，epoll 的特点是：【重要】

1. 使用红黑树存储文件描述符集合，每个文件描述符只需在添加时传入一次，无需用户每次都重新传入。
2. 通过异步 IO 事件找到就绪的文件描述符，而不是通过轮询的方式。



【问题】异步 IO 事件怎么理解？

epoll 使用了多次的 wake up 回调函数机制，当完成时主动通知进程，而不是进程进行轮询。



数据结构如下所示。

~~~c
struct eventpoll {
 wait_queue_head_t wq;      // 等待队列链表，存放阻塞的进程
 struct list_head rdllist;  // 数据就绪的文件描述符都会放到这里
 struct rb_root rbr;        // 红黑树，管理用户进程下添加进来的所有 socket 连接
}
~~~



如下图所示。

![image-20240109203104086](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240109203104086.png)







### 6、epoll_ctl 函数

将 socket 连接注册到 eventpoll 对象里，会做三件事：

1. 先创建一个 epitem 对象。主要包含两个字段，文件描述符和所属的 eventpoll 对象的指针；
2. 将一个数据到达时用到的回调函数添加到 socket 的进程等待队列中。
3. 最后，创建的 epitem 全部放入 eventpoll 的红黑树管理。



【注意】socket 的进程等待队列和阻塞 IO 模式不同的

* epoll 添加的是回调函数，而阻塞 IO 的时进程描述符。因为在 epoll 中，进程是维护在 eventpoll 中的等待队列中了，等待被 epoll_wait 函数唤醒，而不是放在 socket 的进程等待队列中。



如下图所示。

![image-20240110114813899](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240110114813899.png)





### 7、epoll_wait 函数

网卡有数据返回时，eventpoll 的使用过程如下：

1. 通过 socket 里的回调函数【ep_poll_callback 】，从红黑树中找到 epitem。因为 epitem 里面有文件描述符，和 socket 一一对应的。
2. 将找到的 epitem 添加到 eventpoll 的 rdllist。
3. 从 wp 中找到对应的进程。因为 wp 的进程里保存在进程描述符和回调函数【default_wake_func 】
4. 通过进程里的回调函数，唤醒对应的进程。
5. 进程唤醒后，把 rdlist 中就绪的事件返回给用户进程，让用户进程调用拷贝数据到用户空间使用。

可以看到，通过将就绪的 socket 放入在 rdllist中，并且通过 wp 主动通知进程，不在需要进程轮询所有的 socket。

如下图所示。

![image-20240110124802698](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240110124802698.png)


