---
title: 我在生产上遇到了慢查询！
categories: 好的学习笔记
tags: MySQL
---



# MySQL 慢查询优化

生产环境遇到了慢查询，慢查询的 sql 是三表联查，导致接口超时了。

数据量：

* moor_order_tb  **95w**
* moor_call_record_tb **140w**
* moor_call_satisfaction_tb **145w**



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

