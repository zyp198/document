## SQL优化

为什么需要SQL优化？执行时间太长 等待时间太长 链接查询 索引失效 服务器参数设置不对

a SQL 的编写过程

   解析过程

b SQL优化

 索引相当于书籍的目录

索引的优势

索引的劣势

建索引 相当于给该字段的数据排序（二叉排序树）

但是各种索引的类别不一样 比如主键索引 唯一索引 等等 这些索引之间的实际区别是什么呢

![1608702549254](D:\Typora Note\image\1608702549254.png)

![1608703642547](D:\Typora Note\image\1608703642547.png)

**执行计划中的id**

select 查询标记的序列号，用于表示执行的顺序

1、 id相同，执行顺序由上至下

2、 id不同，id值越大优先级越高，越先被执行。

**执行计划中select_type**

查询的类型，主要是用于区分普通查询、联合查询、子查询等。

- **SIMPLE**：简单的 select 查询，查询中不包含子查询或者 union
- **PRIMARY**：查询中包含子部分，最外层查询则被标记为 primary
- **SUBQUERY/MATERIALIZED**：SUBQUERY 表示在 select 或 where 列表中包含了子查询，MATERIALIZED**：**表示 where 后面 in 条件的子查询
- **UNION**：表示 union 中的第二个或后面的 select 语句
- **UNION RESULT**：union 的结果

![1608704662855](D:\Typora Note\image\1608704662855.png)

### 执行计划的 type 

访问类型，SQL 查询优化中一个很重要的指标，结果值从好到坏依次是：system > const > eq_ref > ref > range > index > ALL。

- **system**：系统表，少量数据，往往不需要进行磁盘IO
- **const**：常量连接
- **eq_ref**：主键索引（primary key）或者非空唯一索引（unique not null）等值扫描
- **ref**：非主键非唯一索引等值扫描
- **range**：范围扫描
- **index**：索引树扫描
- **ALL**：全表扫描（full table scan）

**system**

从系统库 MySQL 的系统表 time_zone 里查询数据，访问类型为 system，这些数据已经加载到内存里，不需要进行磁盘 IO，这类扫描是速度最快的。

 **const 扫描的条件为：** 

1. 命中主键（primary key）或者唯一（unique）索引

2. 被连接的部分是一个常量（const）值 

   **Extra**

   **单表优化**

using index （不需要回原表）

using where(需要回表查询)

假设age是索引列

查询sql select age name from ... where age = ..

回表查询name

```sql
添加索引 alter table stu index idx_a1_a2_a3（a1,a2,a3） 复合索引
explain select a1,a2,a3 from stu where a1=1,a2=2,a3=3;推荐写法
explain select a1,a2,a3 from stu where a2=1,a1=2,a3=3;
explain select a1,a2,a3 from stu where a1=2,a3=3; SQL使用了a1索引，这个字段不需要进行回表查询 而由于a3跨列使用，造成了索引失效，因此需要回表查询。
小结：
索引不能跨列使用
保持索引使用的一致性
索引需要逐步优化
将in的范围查询放到
in会使索引失效（通过key_len 验证）
```

### 多表优化及避免索引失效的原则

```java
索引应该给谁加？
结论：小表驱动大表。索引建立在经常使用的字段上。
小表10条 大表3000
for(int i=0;i<10;i++){
	for(int i=0;i<3000;i++){

	}
}
最终会循环30000次

```

