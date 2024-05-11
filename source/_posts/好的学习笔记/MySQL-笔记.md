---
title: 笔记-MySQL
categories: 好的学习笔记
tags: MySQL
---



# MySQL 基础

### ifnull

成功执行：

~~~sql
SELECT
	t.NAME 
FROM
	(
	SELECT
		IFNULL(( SELECT * FROM rp_components_change_tb WHERE order_id = 1000425 ), 0 ) AS 'a',
		CONCAT( 'first' ) AS 'name' UNION
	SELECT
		IFNULL(( SELECT * FROM rp_components_change_tb WHERE order_id = 1000425 ), 0 ) AS 'a',
	CONCAT( 'second' ) AS 'name' 
	) t
~~~



不能执行：

~~~sql
SELECT
	1 
FROM
	(
		( SELECT 1 FROM rp_components_change_tb WHERE order_id = 1000425 ) AS 'a' UNION
	( SELECT 1 FROM rp_components_change_tb WHERE order_id = 1000425 ) AS 'a' 
	)
~~~



### 数据类型 Bigint

bigint 是 int 类型中的一种，一直以来我都以为限制 int 的长度为 11 位，直到碰到这个问题查询相关资料才明白，11 代表的并不是长度，而是字符的显示宽度，在字段类型为 int 时，无论你显示宽度设置为多少，int 类型能存储的最大值和最小值永远都是固定的。

![image-20230329102936741](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102936741.png)



### 内连接

逗号的连表方式就是内连接。

~~~sql
-- join
select * from A  join B on A.id = B.id 

-- inner join
select * from A inner join B on A.id = B.id 

-- 逗号的连表方式就是内连接
select * from A , B where A.id = B.id 
~~~





### 外连接

如果你想要返回左边表的所有数据，那就使用左连接。



【问题】sql 中，匹配条件放在 left join 和 where 的区别

where 子句会在连接后筛选行，只有满足条件的行才会被包括在结果中，当你想保留左表的所有数据，但是右表又需要过滤条件时，就不能把查询条件放在 where 中了，这会把左边的数据也过滤了。

这时候，把右表的查询条件放在 left join 中就能达到效果。



如下例子，为了查询所有推广人，并且计算推广人的所有收益。

1. 将推广表和账单表左连接，根据用户编号
2. 将右表的查询条件放到 left join，保证不被左表的完整。

~~~sql
## 错误例子
SELECT 
    up.*,
    COALESCE(SUM(ubd.consume_amount), 0) AS total_earn
FROM 
    user_promotion_tb up
LEFT JOIN 
    user_bill_details_tb ubd ON up.promotee_id = ubd.user_id
WHERE 
    up.promotion_type = 1
    AND up.line_type = 1
    AND up.promoter_phone = '15177780001'
		AND ubd.type = 1
GROUP BY 
    up.promotee_id
ORDER BY 
    total_earn DESC;
    
## 正确例子
SELECT 
    up.*,
    COALESCE(SUM(ubd.consume_amount), 0) AS total_earn
FROM 
    user_promotion_tb up
LEFT JOIN 
    user_bill_details_tb ubd ON up.promotee_id = ubd.user_id AND ubd.type = 1
WHERE 
    up.promotion_type = 1
    AND up.line_type = 1
    AND up.promoter_phone = '15177780001'
GROUP BY 
    up.promotee_id
ORDER BY 
    total_earn DESC;
~~~





### 数据类型 json

json 类型是从 MySQL 5.7 版本开始支持的功能，而 8.0 版本解决了更新 json 的日志性能瓶颈。如果要在生产环境中使用 json 数据类型，强烈推荐使用 MySQL 8.0 版本。



json 数据类型优势是什么？

- 存储在 json 列中的 json 文档的会被自动验证。无效的文档会产生错误；

  >ERROR 3140 (22032): Invalid json text: "Invalid value." 

- 最佳存储格式。存储在 json 列中的 json 文档会被转换为允许快速读取文档元素的内部格式。

  > 之前没办法针对 json 内的数据进行查询操作，所有的操作必须读取出来 parse 之后进行，非常的麻烦。原生的 json 数据类型支持之后，我们就可以直接对 json 进行数据查询和修改等操作了，较之前会方便非常多。



如何查询json内的数据？

- 【$.path】 用于 JSONObject 类型数据
- 【$[idx]】 用于 JSONArray 类型数据
- 【$】 代表整个 JSON 数据的 root 节点



![image-20230310181411791](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230310181411791.png)



![image-20230310181445775](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230310181445775.png)





### and 和 or

MySQL首先对优先级较高的运算符进行求值。AND 的优先级大于 OR，MySQL会在`AND`运算符之后再对`OR`运算符进行求值。如下所示，先计算 false AND false， 再计算 OR

~~~sql
SELECT true OR false AND false; 
~~~





### order by

~~~sql
select * from product order by 字段A desc,字段B desc
~~~

影响：数据会先按照第一个字段排序（price），如果第一个字段的值相同，再按照第二个字段排序！



如下图所示，先按照 price 排序，后来因为price相同，再按照 order_count 排序

![image-20230404170838166](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230404170838166.png)





### 字段没有默认值，为空会发生什么？

sql1如下，能查询出一条记录，但是 customer_power_status 为 null。

~~~sql
SELECT * FROM `power_exchange_record_tb`
WHERE
        business_no = '1773273698439139328'
        AND platform = 'kst'
        AND bus_status IN (34, 40)
~~~



sql2如下，查询不出记录。因为这与 NULL 进行比较时使用 `!=` 或 `=` 时并不会返回预期的结果。NULL 与任何值的比较都会返回未知（UNKNOWN）

~~~sql
SELECT * FROM `power_exchange_record_tb`
WHEREs
        business_no = '1773273698439139328'
        AND platform = 'kst'
        AND bus_status IN (34, 40)
        AND customer_power_status != 1
~~~





# MySQL explain

### 1、type

* system：这表示查询将只返回一行结果，这是最快的查询类型。
* const：这表示 MySQL 在查询时，通过索引一次就找到了，const 是 const（常数）表的特殊情况，一般出现在主键或唯一索引的查询中。
* eq_ref：连接索引，对于每个索引键，表中只有一条记录与之匹配。
* ref：普通的索引，这个索引可以用来从表中选择某些行。
* range：索引范围扫描，常见于使用【>,<,is null,between ,in ,like等】运算符的查询中。
* index：索引全表扫描，把索引从头到尾扫一遍，常见于使用索引排序或者分组的查询【order by, group by】。
* all：没有命中索引全表扫描，这是最慢的查询类型，应尽量避免。



### 2、Extra

* Using where：使用 WHERE 过滤器，这意味着 MySQL 将在存储引擎层使用 WHERE 条件过滤行。
* Using index：使用覆盖索引，即查询的结果可以直接从索引中获取，而不必读取实际的行。
* Using temporary：使用临时表来处理查询，通常出现在排序操作中。
* Using filesort：文件排序，这可能会导致性能问题，特别是对大数据集的查询。
* Range checked for each record：这表示 MySQL 正在优化查询，但是对每个记录都进行了范围检查。
* Full scan on NULL key：这表示 MySQL 正在执行全表扫描以查找具有NULL键值的行。



![image-20231216155516627](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231216155516627.png)

以上连表查询的分析如下：

* 第一个表是全表查询，因为 type==ALL。扫描的行数是 71，但是实际结果是 12 行。正在使用文件排序，因为 order by 没有符合索引排序规则。
* 第二个表显示 type==ref，并且 possible_keys==userId，ref==promotee_id，它觉得这个字段是索引，并且和表一的 promotee_id 关联，因为表之前的关联条件 userId=promotee_id。



## Using temporary

当使用非驱动表字段进行排序时，就会产生临时表。

## Using filesort

文件排序，虽然是在内存中进行排序，但是相比索引排序也是耗时很多。即使指定了索引字段来进行排序，当数据量少的时候，mysql 优化器也会选择文件排序，而不是索引排序。





# MySQL 和 Redis

### 前提

【问题】都是数据库，使用 redis 作为缓存，数据还需要存入数据库中吗？

* Redis 的持久化机制：A 机器崩溃时可以将 redis 中的数据转移到集群中机器 B 的内存中。那么，数据存入 redis 就可以不用存入mysql，可以直接将 redis 当数据库来用了。查询数据还比 mysql 快。
* 目前还不适合。



### 目前还不适合替代

使用一项技术的时候，不是看它能不能，而是要看它适合不适合；而在大部分场景下，Redis 是无法替代 MySQL 的。目前主流的需求场景下，MySQL 这种关系型数据库还是主流。

MySQL 是关系型数据库，数据储存在磁盘上，数据的格式是我们熟知的二维表格的样式。关系型数据库具有很多强大的功能；大部分都支持 SQL 语句查询，对事务也有很好的支持。

MySQL 和 Redis 没有竞争的关系，通常当并发访问量比较大的时候，特别是读操作很多，架构中可以引入 Redis。



### 总结

所以对于数据，“经常要被查询类”和“缓存类”，可以使用 Redis 来存储减轻访问 MySQL 的压力。注意它只是作为临时存储，所以的数据最终还是存储 MySQL 。





# MySQL 和 Redis 的数据一致性

### 前提

在满足实时性的条件下，不存在两者完全保存一致的方案，只有最终一致性方案。 

[如何保障 MySQL 和 Redis 的数据一致性?](https://mp.weixin.qq.com/s/aHZWwoWIUFz0hzbw3gH4Wg?vid=1688855323406858&deviceid=e49ee5d7-811c-463e-a2d5-92350cd767ad&version=4.1.0.6011&platform=win])



### 1、先写 MySQL，再写 Redis

> * 这是不好的方案
> * 万一 DB 挂了，你把数据写到缓存，DB 无数据，这个是灾难性的；如果写 DB 失败，对 Redis 进行逆操作，那如果逆操作失败呢，是不是还要搞个重试？



![image-20230303102538344](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230303102538344.png)



> - 橘黄色的线是请求 A，黑色的线是请求 B；
> - 橘黄色的文字，是 MySQL 和 Redis 最终不一致的数据；
> - 数据是从 10 更新为 11；





### 2、先写 MySQL，再删除 Redis。

> - 比较推荐这种方式：更新删除，查询回写
> - 删除 Redis 如果失败，可以再多重试几次，否则报警出来；这个方案，是实时性中最好的方案，在一些高并发场景中，推荐这种



情况1：

> 在请求A进行 更新删除的时候，请求A没有进行删除缓存之前请求B查询到的缓存，是旧的。

​	以下这种情况，对于第一次查询，请求 B 查询的数据是 10，但是 MySQL 的数据是 11，**只存在这一次不一致的情况，对于不是强一致性要求的业务，可以容忍。**（那什么情况下不能容忍呢，比如秒杀业务、库存服务等。）

![image-20230303102914077](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230303102914077.png)



情况2：

请求B查询回写的时候，请求B没有进行回写之前，请求A已经更新了 DB，但是进行删除缓存的时候，由于缓存已经失效了，缓存将会是B会写的旧的缓存。

![image-20230303104358037](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230303104358037.png)



但是，这里需要满足 2 个条件：

- 缓存刚好自动失效；
- 请求 B 从数据库查出 10，回写缓存的耗时，比请求 A 写数据库，并且删除缓存的还长。

对于第二个条件，我们都知道更新 DB 肯定比查询耗时要长，所以出现这个情况的概率很小，同时满足上述条件的情况更小。







# MySQL 的排序优化

## 前提

排序是成本很高的操作，但是排序的需求是最频繁出现的，首先推荐的是索引生成排序，然后是业务逻辑排序，最后才是 MySQL 文件排序。



## 排序方式

### 1、索引排序

需要满足以下条件

* 索引的顺序要和 order by 子句的顺序一样，或者符合最左匹配。
* 如果 order by 的第一个字段是常数，剩下的顺序一样。
* 多表连接查询时，order by 子句需要全部属于第一个表。

~~~sql
index(date, order_id, share_id)

##1、索引顺序一样
select *
from order_tb
order by date, order_id, share_id


##2、字段是常数
select *
from order_tb
where date = '2023-12-12'
order by order_id, share_id

##3、多表查询
select *
from order_tb o
inner join share_tb s
order by o.date, o.order_id
~~~



### 2、业务逻辑排序

业务逻辑排序的不足就是，如果数据是需要分页的，那么就只能对部分数据进行分页。最终的结果是不对的。

~~~java
list.sort(Comparator.comparing(UserPromotionTbDO::getPromoterEarn).reversed());
~~~



### 3、mysql 文件排序

文件排序就是，mysql 需要在内存或者磁盘中创建最大空间，用来保存需要的数据列，完成排序后再返回结果。

【问题】怎么理解最大空间

比你的字段是 varchar，那么分配的空间就是其完整的长度。



## 分页排序优化

列表展示，分页和排序通常是一起出现。但是即使使用了 limit，它也是在排序后才应用。

### 1、limit 的问题

使用 limit 头疼的问题就是，在它偏移量很大的时候，查询的无用数据是非常大的，如 limit 1000,20，需要查询 1020 条记录然后只返回 20 条，1000 条被抛弃。



### 2、limit 的优化

查看下面的分页排序优化，

1. film_id 是主键索引，节点存储了实际值，所以能快速查询出已经分了页的记录
2. 前面生成的子表和主表进行连接查询，通过子表去匹配主表，子表上都是需要的值，所以时间上减少了很多。

~~~sql
## 修改前
select film_id , film_description from sakila.film order by title limit 1000,20

## 修改后
SELECT
	film.film_id,
	film_description 
FROM
	sakila.film
	INNER JOIN 
	( SELECT film_id FROM sakila.film ORDER BY title LIMIT 1000, 20 ) AS lim USING (film_id)
~~~





# MySQL 事务隔离级别

### 前提

[Innodb中的事务隔离级别和锁的关系](https://tech.meituan.com/2014/08/20/innodb-lock.html)



### 一次封锁 or 两段锁？

正常一般应用使用的都是一次封锁的方式，比如某个方法需要进行并发安全，则直接对该方法进行加锁。具体哪个方法是我们直接知道的。

但是但在数据库中却不适用，因为这是需要数据库进行加锁，我们知道那些数据需要加锁，但是数据库并不知道会用到哪些数据。所以数据库使用的是两段锁。



### Read uncommitted 和脏读

读写都不加锁，也就是可能读取到其他会话中未提交事务修改的数据——脏读。



### Read Committed

在RC级别中，数据的读取都是不加锁的，但是数据的写入、修改和删除是需要加锁的。如下所示，事务 A 对数据进行 update 时，数据则会被加锁直到事务 A 提交，此时如果事务 B 也对同一数据进行 update 操作，则需要【等待】。

![image-20230531151104759](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230531151104759.png)



### Repeatable Read

可重复读是 MySQL 中 InnoDB 默认的隔离级别。可重复读的意思是，同一事务中，重复读取时可以读取到相同的数据。



如下所示，在 RC 隔离级别下，事务 A 中第一次和第二次查询的结果不一致，因为中间被事务 B 修改了。

![image-20230531151435904](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230531151435904.png)



但是，在 RR 隔离级别下，事务 A 中第一次和第二次查询的结果是一致的，即使其它事务对数据进行了修改。

![image-20230531151649050](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230531151649050.png)



【问题】此时我们是不是有一个疑惑，为什么要保证同一个事务中多次查询数据一致呢，如果我对数据进行了修改，我需要的是最新的数据才对吧？

* 是的，比如说用户 A 和用户 B 都购买了库存为 10 的商品，正常的情况是事务 A 将库存改为了 9，事务 B 查询时获得库存为 9，并且修改为 8。但是按照可重复读，事务 B 查询时获得库存为 10，并且修改为 9。
* 但是，如果你觉得需要让它在事务还没提交的时候，能读取到最新的值，那问题就更大了，如果事务A最终失败回滚了，那事务B已经把库存减少到了8，这不是更糟糕吗？



【问题】锁冲突时等待锁释放还是直接失败？

* 事务可能会在遇到锁冲突时不会立即失败，而不是等待锁释放，直到默认的超时等待时间50s后，会失败回滚。


【问题】可重复读隔离级别下如果保证数据一致性呢？
* 使用悲观锁，for update，牺牲一下当前表的性能。
* 提高隔离级别为串行化级别，牺牲所有的表的性能，呃呃呃呃呃。
* 使用分布式锁，借助第三方吧意味着又要多一份维护，不仅仅是维护系统故障、还有死锁、网络性能等等。
* 修改超时等待时间，锁一冲突直接失败回滚。




### Serializable

读加共享锁，写加排他锁，读写互斥。使用的悲观锁的理论，实现简单，数据更加安全。



### 快照读和当前读

MySQL 为了减少锁处理（包括等待其它锁）的时间，提升并发能力，引入了快照读的概念，使得select不用加锁。而 update、insert 这些“当前读”，就需要另外的模块来解决了。

对于这种读取历史数据的方式，我们叫它快照读 (snapshot read)，而读取数据库当前版本数据的方式，叫当前读 (current read)。很显然，在MVCC中：

- 快照读：就是 select
  - select * from table ….;
- 当前读：特殊的读操作，插入/更新/删除操作，属于当前读，处理的都是当前的数据，需要加锁。
  - select * from table where ? lock in share mode;
  - select * from table where ? for update;
  - insert;
  - update ;
  - delete;





# MySQL 和分布式锁

### 前提

进行多线程编程时，一直疑惑一个问题就是，当我使用锁的时候，数据库事务的范围往往是大于锁的范围，当锁释放了，事务还没有提交，此时其它线程进来了，访问到的数据还是当前事务未提交的旧数据，那么进行旧数据的修改，这不是有问题吗？



如下所示。A 是事务范围，B 是锁的范围，B 范围内使用了锁保证了每次只有一个线程可以范围。

![image-20231019114118788](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231019114118788.png)



伪代码如下所示。

~~~java
@Transactional(rollbackFor = Exception.class)
public void testSynchronized() {
	saveOrUpdate(1);
}

// 方法同步
public synchronized void saveOrUpdate(int userId) {
	User user = UserDao.queryUserById(userId);
	user.setName = "Rambo";
	UserDao.insertUser(user);
}
~~~





### 解决

#### 1、加大锁的范围，使得锁的范围大于事务范围

伪代码如下所示。

~~~java
// 方法同步
public synchronized void testSynchronized() {
	saveOrUpdate(1);
}

@Transactional(rollbackFor = Exception.class)
public void saveOrUpdate(int userId) {
	User user = UserDao.queryUserById(userId);
	user.setName = "Rambo";
	UserDao.insertUser(user);
}
~~~

但是，这样锁的范围太大了，降低了并发性。



#### 2、将数据库隔离级别提升为串行化

因为在更高级时，连 select 都是加锁的，你得等到锁的释放才能读取到数据，前一个事务提交后锁才会释放，这时候数据毫无疑问是最新的了。





# MySQL InnoDB

## 一、前提

[索引](https://cocola6s6.github.io/%E7%B4%A2%E5%BC%95/)

[MySQL InnoDB详解](https://pphc.lvwenhan.com/part-four/detailed-explanation-of-mysql-innodb/section-0)



和之前的 MyISAM 相比，InnoDB 最大的变化就是将磁盘上数据的基本存储结构从索引+数据这样的分体式，变成了所有数据都挂在索引上的整体式。也就是使用 B+ 树为索引数据结构。



## 二、索引

### 1、索引层

数据库数据是存储在磁盘，磁盘顺序 IO 性能比较高，所以数据是以【行和块】为单位，按顺序存储的。

此时，SQL 查询的过程如下：

1. 从磁盘取出所有的数据块。
2. 一行一行地按顺序进行匹配。

所以，虽然顺序 IO 提高了读取速度，但是 IO 的次数还是很多。其中进行了很多无效数据的访问。那么怎么减少无效数据的访问呢？



【问题】那么怎么减少无效数据的访问呢？

答案是，将多个块做成新的块。每个块取一个唯一标识，放到新的块上面。



首先，我们发现在多数情况下，定位操作并不需要匹配整行数据。如果我们每条记录有一个唯一值，那么通过这个唯一值就可以确定这条记录，将所有的唯一值用新的块存储【索引层】，然后用指针关联对应的数据就可以了。通过索引层匹配数据，肯定减少了 IO 次数了。

如下所示。

![image-20231218150933969](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231218150933969.png)



同样的，当数据量愈来愈大时，索引层也会很大。同理，那就把索引层的也当做“记录”来处理，唯一值就是当前索引层的第一条记录的唯一值。

如下所示。

![image-20231218152237579](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231218152237579.png)



### 2、B+ 索引树

上面的图是不是有点熟悉，让我们倒过来看，就是所谓的 B+ 索引树了。如下图所示。

![image-20231218152440162](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231218152440162.png)

此时，SQL 的查询过程变成如下：

> select * from users where id=6

1. 将顶部页的数据读入内存，将与每个 `id|页号` 进行大小对比，找到 6 所在的页。
2. 将数据页读入内存，重复上述比对操作，直到找到最底层的数据页。
3. 将数据页读入内存，找出 `id=6` 索引下挂载的全部数据，这就是所需的这行数据。



### 3、B+ 索引树-页

页是 MySQL 中数据存储的基本单元，默认大小被设置为了 12KB 或者 16KB。

>它的结构是，id|页号，id可能是 int(4字节) 或者 bigint(字节)，但是页号是 int 类型。



### 4、什么时候会进行索引页调整

上面知道，每个页的大小最小是 16KB，所以它存储的内容是有限的。假设每一行数据的大小是 1KB，那么它就只能叶子节点存储的数据到达第 15 条时，一个节点就会被分成三个节点。 

![image-20231218200030786](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20231218200030786.png)



上面说的是叶子节点的情况，叶子节点会保存数据。但是对于非叶子节点来说，它们保存的是页指针，所以它们可以保存更多的“数据”。如下计算：

> 14 * 1024 / 12 =1194 个页指针





# MySQL 回表和覆盖索引

## 一、前提

[MySQL的覆盖索引与回表](https://juejin.cn/post/6844904062329028621)



【问题】什么是回表查询？



## 二、聚簇索引和辅助索引

我们常说的主键，就是聚簇索引，因为它能唯一表示数据。辅助索引就是我们建立的二级索引。如下图所示。

![image-20240125161756877](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240125161756877.png)

【解释】

当我们通过聚簇索引查询数据时，能够通过扫描一次 B+ 树就可以定位到行记录。如下所示。

~~~sql
select * from table where id = 10
~~~



当我们通过辅助索引查询数据时，仔细观察上图可知，辅助索引的叶子节点上保存的是辅助索引的值和对应的行记录的主键，所以，如果只是查询辅助索引值，那么也可以通过扫描一次 B+ 树就定位到行记录。如下所示。

~~~sql
## 假设name是辅助索引
select name from table where name = "name_test"
~~~



但是如果响应查询索引以外的列数据，就需要进行【回表查询】。这个时候，必须通过 2 次 B+ 数才能定位到行数据。

1. 第一次扫描，通过辅助索引先定位到聚簇索引的值
2. 第二次扫描，通过聚簇索引的值定位到要查找的行记录数据

如下所示。

~~~sql
## 先通过辅助索引 name ='name_test' 定位到主键值 id=1
## 再通过聚集索引 id=1 定位到行记录数据
select * from user where name = 'name_test';
~~~



## 三、回表查询

通过辅助索引查不到想要的列数据，只能再通过辅助索引上的聚簇索引，找到完整的行数据，进行了两次索引 B+ 树，它的性能较扫一遍索引树更低。



## 四、索引覆盖

【问题】什么是索引覆盖呢？

* 能在一棵树上把数据找到，就是索引覆盖。



【问题】那实现索引覆盖不是很简单？将要查询的值都建立到同一个索引树上，使用组合索引。

* 确实可以



## 五、组合索引和的单个索引的区别

我们知道，索引就是从数据中抽离出来的一个中间层，它存储在磁盘中。也就是说，对于索引建立得越多，就占用的磁盘空间就越多。

【注意】

这时候你可能会说，我不缺磁盘，多建几个没关系，但是呢，虽然说索引增加了我们查询的速度，我们却不能忽略索引时降低了我们写数据的速度，我们在数据修改时，数据的排布会发生变化，索引的排布当然也会发生变化，所有为什么我们不建议突然大批量操作数据，因为这是一个整体关联的过程。当索引越多，这个整体关联的过程就越多。



【问题】那么组合索引和单个索引的区别是什么呢？

比如，我们建立了组合索引 (a,b,c)，其实是建立了三个索引树，分别是 (a)、(a, b)、(a, b, c)。组合索引生效必须符合【最左匹配原则】，如下所示。

~~~bash
## 生效的
## 注意了，不需要按顺序，mysql会有优化器帮你调整顺序
where a=1
where a=1 and b=1
where a=1 and b=1 and c=1

## 不生效的
where b=1
where c=1
where b=1 and c=1

## 特殊的
### a生效c失效
where a=1 and c=1
### 遇到范围查询(>、<、between、like)，后面的失效。下面两个a、b生效c失效
where a = 1 and b>1 and c=1
where a = 1 and b like 'xxx%' and c=1
~~~



## 六、哪些场景适合使用索引覆盖来优化 SQL

### 1、全表 count 查询优化

count(name) 的操作是，先将所有的 name 列数据查出来到临时表，再进行 count 计算。如果 name 在索引层呢？当然快很多



同理的，下面分页查询也是。

### 2、分页查询





# MySQL group by

## 一、前提

对 group by 的执行不清晰，如下如果我想统计每个人每天的处理时长，应该使用以下 sql ？

~~~sql
SELECT
	handler_operator_id,
	DATE ( reply_time ) AS date,
	SUM(TIMESTAMPDIFF( SECOND, create_time, reply_time )) AS handle_duration 
FROM
	operation_work_order_tb 
GROUP BY
	handler_operator_id;
~~~

执行结果：有日期条件是对的，没有日期是错误的。没有日期时相当于统计的是某个人所有天的处理时长。



~~~sql
SELECT
	handler_operator_id,
	DATE ( reply_time ) AS date,
	SUM(TIMESTAMPDIFF( SECOND, create_time, reply_time )) AS handle_duration 
FROM
	operation_work_order_tb 
GROUP BY
	handler_operator_id,
	DATE ( reply_time );
~~~

执行结果正确。执行结果如下所示。

![image-20240306113821818](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240306113821818.png)



【问题】我是要对用户进行分组，还是要对日期进行分组？

* 要同时对用户和日期进行分组



## 二、分组的原理

分组 sql 的执行顺序如下：

1. 查询并过滤出需要的数据 from test where xxx，生成临时表
2. 对临时表进行 group by



【执行顺序】

* select –> where –> group by–> having –> order by

【问题】having 的作用是什么

* 对分组数据进行过滤。用于对 where 和 group by 查询出来的分组经行过滤，查出满足条件的分组结果。



### 1、group by 单个字段

group by name 的执行如下图所示。

name 是 groupBy 项所以是合并为一个单元格，对于 id 和 number 里面的单元格有多个数据的情况怎么办呢？答案就是用聚合函数，聚合函数就用来输入多个数据，输出一个数据的。如cout(id)，sum(number)，而每个聚合函数的输入就是每一个多数据的单元格。

![image-20240306100920425](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240306100920425.png)





### 2、group by 多个字段

group by 多个字段该怎么理解呢？我们可以把所有字段看成一个整体字段，以他们整体来进行分组。

group by name, number 的执行如下图所示。

![image-20240306101609695](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240306101609695.png)



## 三、总结

回到最开始的问题，应该选择哪一个 sql，哪个 sql 是正确的？答案是两个都能满足需求，但是正确的写法应该是第二个，因为是需要统计某人某天的处理时长，也就是需要按照用户和日期同时进行分组。



【问题】为什么第一个 sql 也能满足需求呢？ 

关键在于查询条件是日期。日期作为查询条件时，它的思路就是，先将符合的日期的所有数据。然后对用户进行分组，然后聚合计算时长。



【注意】当我们需要计算，需要使用分组并且需要使用聚合函数计算的时候，我们只需要也只能去关注需要计算的字段，如上我们只需关注时长字段，因为日期字段是错误的。



【问题】为什么说日期字段是错的呢？

我们使用 group by 的时候，意味着我们肯定会分组合并。对于日期而言，我们无法用聚合函数进行合并，所以合并时它只会默认取第一条。









# MySQL 慢查询优化

生产环境遇到了慢查询，慢查询的 sql 是三表联查，导致接口超时了。

数据量：

* moor_order_tb  **95w**
* moor_call_record_tb **140w**
* moor_call_satisfaction_tb **45w**



sql 如下所示。

~~~sql
EXPLAIN SELECT
	o.id,
	c.exten,
	o.create_time
FROM
	moor_order_tb AS o
	LEFT JOIN moor_call_record_tb AS c ON o.call_id = c.call_sheet_id
	LEFT JOIN moor_call_satisfaction_tb AS s ON o.call_id = s.call_sheet_id 
WHERE
	c.connect_type = 'normal'
    AND o.create_time >= '2024-03-07 00:00:00' 
	AND o.create_time <= '2024-04-07 23:59:59' 
ORDER BY
	o.create_time DESC
~~~



**执行速度：>10s**

![image-20240408225209416](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240408225209416.png)



分析执行，如下所示。

![image-20240408234656414](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240408234656414.png)

问题：

* 没有命中索引全表扫描
* 产生了临时表
* 140w 数据文件排序



原因：

* 索引上限数量 5 个，所以没有为 c.connect_type 建立索引
* 虽然使用了 left join，但是优化器还是选择了 moor_call_record_tb 为驱动表。使用非驱动表字段排序会产生临时表。
* 驱动表上排序了，那肯定得文件排序

可以得出，以上 sql 慢查询的原因就是因为进行了文件排序，对 140w 的结果集进行了文件排序。



## 慢查询的优化思路

1. where 条件选择区分度高的列，这样锁定的数据量少，速度就快。
2. order by limit 形式的 sql 语句让排序的表先查。
3. 加索引。



第一个方案不适合当前场景，因为他必须这个字段要加搜索条件。第三个方案也不适合当前场景，因为规定了索引数量已经够了，不能再加索引了。所以只能在第二个方案上进行，否则就无法进行优化。



## 优化列表 sql

### 先排序再连接

对 140w 的数据进行文件排序，很慢。换个思路，先让它命中索引进行索引排序，排完序后再 join 筛选数据。

如下所示。

~~~sql
explain SELECT
	call_id 
FROM
	moor_order_tb o 
WHERE
	EXISTS (
	SELECT
		1 
	FROM
		moor_call_record_tb c
		LEFT JOIN moor_call_satisfaction_tb s ON c.call_sheet_id = s.call_sheet_id 
	WHERE
		o.call_id = c.call_sheet_id 
		AND c.connect_type = 'normal' 
	)
	AND o.create_time >= '2024-03-07 00:00:00' 
	AND o.create_time <= '2024-04-07 23:59:59' 
ORDER BY
	o.create_time DESC
~~~



**执行速度：2ms**

![image-20240408225041424](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240408225041424.png)



分析执行， 如下所示。

![image-20240408234552509](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240408234552509.png)

* 命中了索引范围查询
* 索引排序



到这里还没结束，列表需要返回其它表的字段，但是优化后的 sql 只能返回 moor_order_tb 的数据。怎么处理呢，可以在业务处理，业务代码上再通过“外键call_id”查询对应想要的其它表的数据，命中了索引，查询很快的，毫秒级别。



## 优化列表总数 sql

### PageHelper 的 select count(*) 耗时严重

在数据库中执行优化后的列表 sql，明明很快的呀，但是代码执行却还是很久很久。最后发现是 PageHelper 分页插件默认执行的 select count(0) 耗时。

如下所示。

sql 耗时 9252ms ，光是 count 就耗时 9025ms。

![img](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/%25E4%25BC%2581%25E4%25B8%259A%25E5%25BE%25AE%25E4%25BF%25A1%25E6%2588%25AA%25E5%259B%25BE_1712545797923.png)



![img](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/%25E4%25BC%2581%25E4%25B8%259A%25E5%25BE%25AE%25E4%25BF%25A1%25E6%2588%25AA%25E5%259B%25BE_17125492521346.png)



### 动态 sql 处理表连接

原来的 count sql，无论什么查询条件都三表联查。所以可以修改为需要的时候再 join。

修改前如下所示。

~~~sql
SELECT
	count(*) 
FROM
	moor_order_tb AS o
	LEFT JOIN moor_call_record_tb AS c ON o.call_id = c.call_sheet_id
	LEFT JOIN moor_call_satisfaction_tb AS s ON o.call_id = s.call_sheet_id
~~~



修改后如下所示。

~~~sql
select count(*)
        from moor_order_tb as o
        <if test="(connectType != null and connectType.size > 0)
            || (callTimeLength != null and callTimeLength!='')
            || (exten != null and exten.size > 0)
            || (agentName != null and agentName.size > 0)
            || (queueName != null and queueName.size > 0)
            || (startTime != null)
            || (endTime != null)
            || (hangupUser != null and hangupUser.size > 0)
            || (offeringTimeStartTime != null and offeringTimeStartTime!='')
            || (offeringTimeEndTime != null and offeringTimeEndTime!='')
            || (callNo != null and callNo!='')
            || (calledNo != null and calledNo!='')">
            left join moor_call_record_tb as c on o.call_id = c.call_sheet_id
        </if>
        <if test="surveyContent != null">
            left join moor_call_satisfaction_tb as s on o.call_id = s.call_sheet_id
        </if>
~~~



**执行速度：65ms**

![image-20240408232518594](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20240408232518594.png)



## 总结

任何数据库层面的优化都抵不上应用系统的优化。如果能合理设计表结构，将可能的排序条件设计在同一个表。如果能合理选择筛选条件。就不会有这么多的慢查询。



参考如下：

[美团技术团队-MySQL索引原理及慢查询优化](https://tech.meituan.com/2014/06/30/mysql-index.html)

[mysql驱动表的选择](https://blog.csdn.net/u013065023/article/details/54964275)

[EXISTS子查询的执行过程](https://juejin.cn/s/%E7%AE%80%E8%BF%B0EXISTS%E5%AD%90%E6%9F%A5%E8%AF%A2%E7%9A%84%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B)

[mysql count函数与分页功能极限优化](https://learnku.com/articles/53559)

