### 参考文档

[基础知识](https://snailclimb.gitee.io/javaguide/#/docs/database/MySQL)

[MySql存储问题](https://juejin.im/post/5b55b842f265da0f9e589e79)

### 补充资料

[InnoDB存储1](https://mp.weixin.qq.com/s?__biz=MzIxNTQ3NDMzMw==&mid=2247483678&idx=1&sn=913780d42e7a81fd3f9b747da4fba8ec&chksm=979688eca0e101fa0913c3d2e6107dfa3a6c151a075c8d68ab3f44c7c364d9510f9e1179d94d&scene=21#wechat_redirect)

[InnoDB存储2](https://mp.weixin.qq.com/s?__biz=MzIxNTQ3NDMzMw==&mid=2247483670&idx=1&sn=751d84d0ce50d64934d636014abe2023&chksm=979688e4a0e101f2a51d1f06ec75e25c56f8936321ae43badc2fe9fc1257b4dc1c24223699de&scene=21#wechat_redirect)

### 索引

MyISAM ：B+Tree叶节点的data域存放的是数据记录的地址。在索引检索的时候，首先按照B+Tree搜索算法搜索索引，如果指定的Key存在，则取出其 data 域的值，然后以 data 域的值为地址读取相应的数据记录。这被称为“非聚簇索引”。

InnoDB ：其数据文件本身就是索引文件，表数据文件本身就是按B+Tree组织的一个索引结构，树的叶节点data域保存了完整的数据记录。这个索引的key是数据表的主键，因此InnoDB表数据文件本身就是主键索引，这被称为==聚簇索引（或聚集索引）==。**聚集索引即索引结构和数据一起存放的索引。主键索引属于聚集索引。**

**在根据主索引搜索时，直接找到key所在的节点即可取出数据；在根据辅助索引查找时，则需要先取出主键的值，再走一遍主索引。** **因此，在设计表的时候，不建议使用过长的字段作为主键，也不建议使用非单调的字段作为主键，这样会造成主索引频繁分裂。**

**主键索引**

> **数据表的主键列使用的就是主键索引。**
>
> **一张数据表有只能有一个主键，并且主键不能为null，不能重复。**
>
> **在mysql的InnoDB的表中，当没有显示的指定表的主键时，InnoDB会自动先检查表中是否有唯一索引的字段，如果有，则选择该字段为默认的主键，否则InnoDB将会自动创建一个6Byte的自增主键。**

**二级索引(辅助索引)**

> **二级索引又称为辅助索引，是因为二级索引的叶子节点存储的数据是主键。也就是说，通过二级索引，可以定位主键的位置。**
>
> 唯一索引，普通索引，前缀索引等索引属于二级索引。



==上诉两种类型，分别属于下面两种==

**聚集索引**

> **聚集索引即索引结构和数据一起存放的索引。主键索引属于聚集索引。**



**非聚集索引**

> **非聚集索引即索引结构和数据分开存放的索引。**
>
> **二级索引属于非聚集索引。**
>
> MYISAM引擎的表的.MYI文件包含了表的索引， 该表的索引(B+树)的每个叶子非叶子节点存储索引， 叶子节点存储索引和索引对应数据的指针，指向.MYD文件的数据。
>
> **非聚集索引的叶子节点并不一定存放数据的指针， 因为二级索引的叶子节点就存放的是主键，根据主键再回表查数据。**







### MVCC 多版本并发控制

[知识](https://segmentfault.com/a/1190000012650596)

[谈谈你对Mysql的MVCC的理解](https://baijiahao.baidu.com/s?id=1629409989970483292&wfr=spider&for=pc)

### 锁

测试：RR隔离级别下，InnoDB是否自己防止了幻读的发生

已知条件：

1、InnoDB只有通过**索引条件**检索数据**才使用行级锁**，否则，InnoDB将使用**表锁**

2、对于`UPDATE、DELETE、INSERT`语句，**InnoDB**会**自动**给涉及数据集加排他锁（X)

3、间隙锁只会在`Repeatable read`隔离级别下使用

4、在默认的情况下，`select`是不加任何行锁的~事务可以通过以下语句显示给记录集加共享锁或排他锁。

- 共享锁（S）：`SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE`。
- 排他锁（X)：`SELECT * FROM table_name WHERE ... FOR UPDATE`。

5、当我们**用范围条件检索数据**而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给**符合范围条件的已有数据记录的索引项加锁**；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP)”。InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁。

例子：假如emp表中只有101条记录，其empid的值分别是1,2,...,100,101

即`select * from emp where empid > 100 for update;`

上面是一个范围查询，InnoDB**不仅**会对符合条件的empid值为101的记录加锁，也会对**empid大于101（这些记录并不存在）的“间隙”加锁**。本例子就是显示的给不存在的记录加了锁

6、InnoDB实现的`Repeatable read`隔离级别配合GAP间隙锁已经避免了幻读！

验证例子：事务A中 select 查询范围，未提交；事务 B 中 INSERT 按上述条件是会给插入的记录加锁的，事务 A 再次 select 查询范围 看结果相同否

**InnoDB确实在RR级别下自己解决了幻读。**

==注意==：事务A，B在不同会话，A处于`read uncommitted`,B处于`repeatable read`，那么B事务会按照自己的隔离级别，读取该读的数据，而A事务能够读取到B事务在未提交的情况下的所有更改，上诉已知条件2中加的锁对事务A来讲没用，事务A依然能够读取到数据

**某事务给某条行记录添加排它锁，其他事务select依然可以查询，只是说无法在这条记录上再加锁了；也就是说select（不显示添加锁的情况下，它默认本身也是不给记录加锁的）在任何事务中都可以直接执行。如果某事务加排它锁，其他事务要等到它提交才能继续加锁（让自己做修改操作）**

==事务中添加的锁在事务提交后才会释放，事务A修改某条数据未提交，事务B修改只能等待事务A提交后才能执行==

**注意**

**间隙锁在InnoDB的唯一作用就是防止其它事务的插入操作，以此来达到防止幻读的发生，所以间隙锁不分什么共享锁与排它锁。如果InnoDB扫描的是一个主键、或是一个唯一索引的话，那InnoDB只会采用行锁方式来加锁，而不会使用Next-Key Lock的方式，也就是说不会对索引之间的间隙加锁**

**InnoDB的可重复读并不保证避免幻读，需要应用使用加锁读来保证。而这个加锁度使用到的机制就是next-key locks。**



InnoDB除了通过范围条件加锁时使用间隙锁外，如果使用相等条件（或者没有条件，可以理解为每行都加了锁）请求给一个不存在的记录加锁（update更新数据时），InnoDB也会使用间隙锁！，当然修改的如果是存在的记录，对于大于当前记录索引的**不存在记录**会有间隙锁，间隙锁锁的是不存在记录。分清楚间隙锁加在了哪，行锁又加在了哪。

**行锁（Record Lock）**：锁直接加在索引记录上面。
**间隙锁（Gap Lock）**：锁加在不存在的空闲空间，可以是两个索引记录之间，也可能是第一个索引记录之前或最后一个索引之后的空间。
**Next-Key Lock**：行锁与间隙锁组合起来用就叫做Next-Key Lock。

**在可重复读隔离级别下，发生幻读的情况**，select 加共享锁或者独占锁，都会使用间隙锁（除去上述不加间隙锁的条件），select 默认是不会加间隙锁的，这种情况下，其他事务能够执行插入操作，其他事务不管提没提交，当前事务再次执行select都只会得到与上一次相同的查询结果，但是如果其他事务插入数据并且提交，当前事务修改了其他事务提交的数据，执行select就会看到其他事务提交的新数据了



### 两类丢失

> 对于`UPDATE、DELETE、INSERT`语句，**InnoDB**会**自动**给涉及数据集加排他锁（X)，这是行级锁

一旦加锁就不存在下列的丢失了情况了，如果使用行级锁就可以针对不同记录修改，不需要等待事务提交，如果使用表级锁那么就无法修改不同记录了，必须等待提交。

回滚丢失（事务A修改，未提交，事务B修改，处于阻塞，事务A回滚，事务B会正常在原数据上修改）

覆盖丢失（事务A修改，未提交，会给该记录加锁（假设使用的是行级锁），事务B的修改该记录操作会等待A提交后才会执行）

### [utf8和utf8mb4的区别](https://www.cnblogs.com/cuiqq/p/11045487.html)

### 什么时候会用索引？

简单分析下

`select *` 且`where`条件没有用索引字段，或者索引字段不符合最左匹配原则，该语句不走索引,where使用索引字段，且符合最左匹配原则就走索引。

`select [索引字段] from ... where [索引字段]`，where中的索引字段，不论是否符合最左匹配原则，都能够走索引，

例如`select city from t where age = 11`,索引字段为(name,age) 不走索引

```mysql
//test_index(id,name,age,city) id主键，(name,age)索引

select [*|city] from test_index where name = 'f';//走索引
select [*|city] from test_index where age = 11;//不走索引

select [name|age|id] from test_index where age = 11;//即使不符合最左匹配原则也走索引

select [id|name|age] from test_index;//走索引
select [*|city] from test_index;//不走索引

//
select [id|name|age] from test_index where age like '1%';//走索引
select [*|city] from test_index where age like '1%';//不走索引

select [*|city] from test_index where name like 'f%';//走索引，符合最左匹配原则

select [*|city] from test_index where name like '%f%';//有左模糊不会走索引
select name from test_index where name like '%f%';//走索引

//===查询结果的字段只有索引字段，必然会走索引==


//===联合索引(a,b,c) 在查询条件a=1 and b>3 and c=4时只会用a,b来匹配索引


```

### MySQL语句问题

清空数据`truncate 表名`比`delete from 表名`更好，前者可以将数据库自增清除到0开始

- InnoDB 使用 delete 清空表，数据库重启，自增字段会从1开始，因为InnoDB 自增量是存储在内存中的
- MYISAM 使用 delete 清空表，自增量存储在文件中，重启仍然从上一次自增量开始

### 补充小知识

*Latin1*是ISO-8859-1的别名,不支持中文

[一条sql执行的很慢的原因](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485185&idx=1&sn=66ef08b4ab6af5757792223a83fc0d45&chksm=cea248caf9d5c1dc72ec8a281ec16aa3ec3e8066dbb252e27362438a26c33fbe842b0e0adf47&token=79317275&lang=zh_CN#rd)

[一条sql如何执行](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485097&idx=1&sn=84c89da477b1338bdf3e9fcd65514ac1&chksm=cea24962f9d5c074d8d3ff1ab04ee8f0d6486e3d015cfd783503685986485c11738ccb542ba7&token=79317275&lang=zh_CN#rd) 有binlog和redolog的讲解

mvcc要使用undolog完成回滚和事务的隔离

[字符集](https://www.cnblogs.com/geaozhang/p/6724393.html#MySQLyuzifuji)