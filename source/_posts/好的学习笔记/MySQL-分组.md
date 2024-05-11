---
title: MySQL分组还是很模糊？
categories: 好的学习笔记
tags: MySQL
---



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
