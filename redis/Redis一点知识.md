Redis 特殊的数据结构

Redis 配置文件

Redis 哨兵模式

Redis 和 MySql 一致性解决方案

Redis 持久化 RDB AOF

Redis底层数据结构使用的skiplist结构，需要学习

### Linux 安装

1、下载安装包

2、解压 `tar -zxcf 压缩包名`

3、基本环境安装 gcc-c++ (C++编译器)

```shell
//切换到解压后的文件目录下

yum install gcc-c++ 

make

make install

//如果安装过程出现‘struct redisServer’没有名为‘maxmemory’的成员 问题
//参考https://blog.csdn.net/happyzwh/article/details/106373688
```



4、更改配置文件

```
# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes //no改为yes进入后台运行，否则会让窗口进入redis-cli界面

```

5、启动服务

```
[root@localhost /]# redis-server /usr/local/bin/redisconfig/redis.conf (指定配置文件)

redis-cli 进入客户端
```

6、查看redis 进程

```
[root@localhost /]# ps -ef|grep redis
root       6869      1  0 18:58 ?        00:00:00 redis-server 127.0.0.1:6379
root       6877   6394  0 19:00 pts/0    00:00:00 grep --color=auto redis

```

7、退出

```
[root@localhost /]# redis-cli 
127.0.0.1:6379> SHUTDOWN
not connected> exit
[root@localhost /]# 

```



8、压力测试

```
Usage: redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests]> [-k <boolean>]

 -h <hostname>      Server hostname (default 127.0.0.1)
 -p <port>          Server port (default 6379)
 -s <socket>        Server socket (overrides host and port)
 -a <password>      Password for Redis Auth
 -c <clients>       Number of parallel connections (default 50)
 -n <requests>      Total number of requests (default 100000)
 -d <size>          Data size of SET/GET value in bytes (default 2)
 --dbnum <db>       SELECT the specified db number (default 0)
 -k <boolean>       1=keep alive 0=reconnect (default 1)
 -r <keyspacelen>   Use random keys for SET/GET/INCR, random values for SADD
  Using this option the benchmark will expand the string __rand_int__
  inside an argument with a 12 digits number in the specified range
  from 0 to keyspacelen-1. The substitution changes every time a command
  is executed. Default tests use this to hit random keys in the
  specified range.
 -P <numreq>        Pipeline <numreq> requests. Default 1 (no pipeline).
 -q                 Quiet. Just show query/sec values
 --csv              Output in CSV format
 -l                 Loop. Run the tests forever
 -t <tests>         Only run the comma separated list of tests. The test
                    names are the same as the ones produced as output.
 -I                 Idle mode. Just open N idle connections and wait.
```

### 基本知识

```shell
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16   # 默认数据库的数量，可以通过select更改

```

一些命令

```bash
key * 显示当前数据库所有键
flushdb 清空当前数据库
flushall 清空所有数据库
exists 是否存在某个key
expire key seconds 设置key存活时长
ttl 查看key剩余存活时长
type key 查看数据类型

```

**Redis 单线程**

基于内存操作，CPU不再是性能瓶颈，内存和网络宽带才是瓶颈

C 语言写的，官方提供的数据10，0000+的QPS 不比memecahe差

**为啥Redis 单线程还这么快？**

误区：高性能服务器一定是多线程？多线程一定比单线程效率高

redis 用的是内存来放数据，对于内存数据来讲，没有上下文切换，效率肯定是最好的，多线程会上下文切换耗费时间。（多线程有效率是在某个线程进行I/O去了(将硬盘数据读入到内存)，将CPU给其他线程使用，避免CPU浪费，而redis的数据都在内存中，直接让CPU执行操作就行了，这样更快）



[redis6.0开始支持多线程处理](https://www.cnblogs.com/mr-wuxiansheng/p/12884356.html)

### 数据结构

> It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, geospatial indexes with radius queries and streams. Redis has built-in replication, Lua scripting, LRU eviction, transactions and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster. 

#### String

```bash
127.0.0.1:6379> set name liang
OK
127.0.0.1:6379> APPEND name ",hello" #字符串追加，如果key不存在就等价于set key
(integer) 11
127.0.0.1:6379> get name
"liang,hello"
127.0.0.1:6379> STRLEN name # 获取长度
(integer) 11
#=======================================
127.0.0.1:6379> set views 0
OK
127.0.0.1:6379> INCR views  # 自增 1 后返回值
(integer) 1
127.0.0.1:6379> INCR views
(integer) 2
127.0.0.1:6379> INCR views
(integer) 3
127.0.0.1:6379> INCR views
(integer) 4
127.0.0.1:6379> DECR views # 自减1后返回值
(integer) 3
127.0.0.1:6379> DECR views
(integer) 2
127.0.0.1:6379> INCRBY views 10 # 按指定步长增加
(integer) 12
127.0.0.1:6379> DECRBY views 2# 按指定步长减少
(integer) 10
#======================
127.0.0.1:6379> GETRANGE name 0 3 # 获取指定子串[0,3]
"lian"
127.0.0.1:6379> GETRANGE name 0 -1  #-1是结尾，等价于get
"liang,hello"
127.0.0.1:6379> SETRANGE name 1 ff # 指定位置开始替换
(integer) 11
127.0.0.1:6379> get name
"lffng,hello"
#====================
# setex   set with expire time
# setnx   set if  key not exists
127.0.0.1:6379> setex key1 30 "ff"
OK
127.0.0.1:6379> get key1
"ff"
127.0.0.1:6379> ttl key1
(integer) 16
127.0.0.1:6379> ttl key1
(integer) 15
127.0.0.1:6379> setnx myname "liang"
(integer) 1
127.0.0.1:6379> setnx myname "liang1"
(integer) 0
127.0.0.1:6379> get myname
"liang"
127.0.0.1:6379> get key1
(nil)
127.0.0.1:6379> 
# =================================
127.0.0.1:6379> mset k1 v1 k2 v2 # 批量设置
OK
127.0.0.1:6379> mget k1 k2 k3 #批量获取
1) "v1"
2) "v2"
3) (nil)
127.0.0.1:6379> msetnx k1 v4 k3 v3 # 原子操作，要么都成功，要么都失败
(integer) 0
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> get k3
(nil)
#=================
set user:1 {name:laing,age:1}  #存对象， user:1是key ,{name:laing,age:1}是json字符串

mset user:1:name liang user:1:age 12 #user:1:name,user:1:age 给这两个key设置值
mget user:1:name user:1:age

#=================

127.0.0.1:6379> getset db mysql #先获取再设置值 ，不存在就返回nil
(nil)
127.0.0.1:6379> get db
"mysql"
127.0.0.1:6379> getset db redis #存在就返回旧值，设置新值
"mysql"
127.0.0.1:6379> get db
"redis"


```

**应用场景：value除了是字符串还可以是数字**

- 计数器
- 统计多单位 例如：uid:1234772:follow   123 ，前面是key,后面是value
- 粉丝数
- 对象缓存

#### List

作用：栈，队列，阻塞队列

所有的命令都是L开头

```bash
127.0.0.1:6379> LPUSH list one
(integer) 1
127.0.0.1:6379> LPUSH list two
(integer) 2
127.0.0.1:6379> LPUSH list three
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> LRANGE list 0 1
1) "three"
2) "two"
127.0.0.1:6379> RPUSH list mmm  # 右边插入
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "mmm"
#===================
RPOP  #右出
LPOP #左出
#=================
127.0.0.1:6379> LINDEX list -1  # 指定index获取值
"mmm"
127.0.0.1:6379> LINDEX list 0
"three"
#============
LLEN #获取list长度
lrem # 移除
127.0.0.1:6379> LRANGE list 0 -1
1) "start"
2) "three"
3) "two"
4) "one"
5) "mmm"
6) "end"
7) "start"
127.0.0.1:6379> LREM list 2 start  # 从头开始删除，删除指定个数的指定的值
(integer) 2
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "mmm"
5) "end"
#===================
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "mmm"
5) "end"
127.0.0.1:6379> ltrim list 1 2 #保留指定范围的数据，其余的删除
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
#===============
rpoplpush list1 list2 # 从list1弹出，然后从左边进入list2
lset list 0 value # 给指定list的指定Index设置value，如果list不存在或者index非法就返回error

127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
127.0.0.1:6379> LINSERT list before one three #指定value的后面插入
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "three"
3) "one"
127.0.0.1:6379> LINSERT list after one three
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "three"
3) "one"
4) "three"



```

**小结**

- 左右两边插入效率高，中间低
- 做消息队列

#### Set

```bash
sadd key value1 value2 #放值,可以放多个
smembers key #获取所有的值
sismember key value #指定值是否存在
scard key#获取集合中的个数
srem key value#移除指定元素
srandmember key#从集合中随机抽出一个元素
spop key#随机删除一个元素
smove set1 set2 value#将set1的value移到set2
sdiff set1 set2 #set1集合减去set1和set2的交集
sinter set1 set2 #交集
sunion set1 set2 #并集
```

微博，关注的所有人放入set集合

共同好友，共同关注

#### Hash

存储的值是Map集合

```bash
hset myhash key value
127.0.0.1:6379> HSET myhash k1 v1 #设置单个k,v
(integer) 1
127.0.0.1:6379> HGET myhash k1
"v1"
127.0.0.1:6379> HMSET myhash k2 v2 k3 v3 #批量设置k,v
OK
127.0.0.1:6379> HMGET myhash k1 k2 k3
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> HGETALL myhash
1) "k1"
2) "v1"
3) "k2"
4) "v2"
5) "k3"
6) "v3"
127.0.0.1:6379> HSET myhash k1 v4
(integer) 0
127.0.0.1:6379> hget myhash k1
"v4"
127.0.0.1:6379> HGETALL myhash #按键值显示所有数据
1) "k1"
2) "v4"
3) "k2"
4) "v2"
5) "k3"
6) "v3"
hdel myhash k1 #删除指定看k,v
hlen myhash #获取hash表长度
hexists myhash k1#判断指定k是否存在
hkeys myhash #获取所有k
hvals myhash #获取所有v
#======================
127.0.0.1:6379> hset myfield k1 3
(integer) 1
127.0.0.1:6379> HINCRBY myfield k1 2 #步长增加指定值，步长为负数就是减法
(integer) 5
127.0.0.1:6379> HGET myfield k1
"5"
127.0.0.1:6379> HSETNX myfield k2 6 #不存在就新增k,v
(integer) 1
127.0.0.1:6379> HSETNX myfield k2 7 #存在就不能新增
(integer) 0
127.0.0.1:6379> hget myfield k2
"6"

```



**小结：**

hash更适合存储对象，经常变更的数据，String 相比就不太适合了

hmset user:1 name liang age 12,存储用户信息

#### Zset (Sorted Set)

排序的set

```bash
127.0.0.1:6379> zadd myzset 1 one  #根据数字排顺
(integer) 1
127.0.0.1:6379> zadd myzset 2 two 3 three
(integer) 2
127.0.0.1:6379> zrange myzset 0 -1 #升序
1) "one"
2) "two"
3) "three"
ZREVRANGE myzset 0 -1#降序
#==============================
127.0.0.1:6379> zadd salary 500 zhangsan 2000 lisi 2500 hong
(integer) 3
127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf #按score，升序排列，-inf +inf 负无穷到正无穷
1) "zhangsan"
2) "lisi"
3) "hong"
127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf withscores #按score，升序排列，显示score
1) "zhangsan"
2) "500"
3) "lisi"
4) "2000"
5) "hong"
6) "2500"
127.0.0.1:6379> ZRANGEBYSCORE salary -inf 2000 withscores #小于等于2000的升序
1) "zhangsan"
2) "500"
3) "lisi"
4) "2000"
127.0.0.1:6379> ZREVRANGEBYSCORE salary 2000 500 withscores #降序
1) "lisi"
2) "2000"
3) "zhangsan"
4) "500"

zrem myset v#移除指定v
zcard myset #获取集合元素个数
zcount myset from to #获取score在[from,to]区间的个数
```

**小结：**

排行榜应用

成绩，工资排名



### 三种特殊结构

#### geospatial 地理位置

用处：朋友定位，附近的人，打车距离计算

推算两地距离，方圆几里的人

> 一般通过java程序导入城市数据
>
> 参考https://redis.io/commands/geohash

```bash
# Valid longitudes are from -180 to 180 degrees. 经度
# Valid latitudes are from -85.05112878 to 85.05112878 degrees. 纬度
GEOADD key longitudes latitudes member #可以添加多个城市
GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
#=======================
GEODIST #获取距离
m for meters.
km for kilometers.
mi for miles.
ft for feet.
redis> GEODIST Sicily Palermo Catania
"166274.1516"
redis> GEODIST Sicily Palermo Catania km
"166.2742"
redis> GEODIST Sicily Palermo Catania mi
"103.3182"
#===============

GEOPOS Sicily Palermo Catania NonExisting #获取多个地点的位置
1) 1) "13.36138933897018433"
   2) "38.11555639549629859"
2) 1) "15.08726745843887329"
   2) "37.50266842333162032"
3) (nil)
#==================
GEORADIUS #以某个经纬度为中心，固定半径查找范围内的地点

redis> GEORADIUS Sicily 15 37 200 km WITHDIST #经纬度（15,37）半径 200 km内的地点集合
1) 1) "Palermo"
   2) "190.4424" #距离信息
2) 1) "Catania"
   2) "56.4413"
redis> GEORADIUS Sicily 15 37 200 km WITHCOORD #显示经纬度
1) 1) "Palermo"
   2) 1) "13.36138933897018433"
      2) "38.11555639549629859"
2) 1) "Catania"
   2) 1) "15.08726745843887329"
      2) "37.50266842333162032"
redis> GEORADIUS Sicily 15 37 200 km WITHDIST WITHCOORD 
1) 1) "Palermo"
   2) "190.4424"
   3) 1) "13.36138933897018433"
      2) "38.11555639549629859"
2) 1) "Catania"
   2) "56.4413"
   3) 1) "15.08726745843887329"
      2) "37.50266842333162032"
#====================
GEORADIUSBYMEMBER #以某个在集合中的成员为中心找其他地点
GEORADIUSBYMEMBER key member radius m|km|ft|mi
#===================
GEOHASH 返回一个代表位置的字符串
Return valid Geohash strings representing the position of one or more elements 
redis> GEOHASH Sicily Palermo Catania
1) "sqc8b49rny0"
2) "sqdtr74hyu0"
#==================
geo底层是zset,可以使用zset去执行一些删除操作
```

#### HyperLogLogs

做基数统计，有点像set，存放不同的元素

优点是占用空间固定位12KB，可以放2^64个元素

0.81%的错误率，可以接受

> 网页的UV，用户访问量，同一个用户不做重复统计，用 set 的话存的相当于是用户id，但是实际上我们更关注的是个数，存放 id 会浪费空间，这个数据结构只需要占用固定大小即可，

```bash
pfadd mykey a b c #创建一组数据
pfcount mykey #统计个数
pfadd mykey2 b c d f g
pfmerge mykey3 mykey mykey2#合并mykey mykey2为新的mykey3
pfcount mykey3 #看并集的数量
```

#### Bitmaps

位存储

统计用户信息，活跃，不活跃，登陆多少人，未登录多少人，打卡，未打卡

只有两个状态

```bash
#存储一周打卡数据
setbit sign 0 1
setbit sign 1 0
setbit sign 2 0
setbit sign 3 0
setbit sign 4 0
setbit sign 5 1
setbit sign 6 1

getbit sign 6 #获取这天的打卡数据

#统计打卡天数
bitcount sign start end #可以[start,end]范围统计，不加这两个，就全范围统计
```



### 事务

==Redis事务不保证原子性，单条命令保证原子性，没有隔离级别概念，就是不会说并发事务执行==

Redis事务：一次性（一次执完所有命令），顺序性，排他性

一个事务中的所有命令都会被序列化，在事务执行过程中，按照顺序执行

```
--- 队列 set set set 执行---
```

所有的命令在事务中先放着，等到发起执行命令时才执行

redis 事务

- 开启事务（multi）
- 命令入队
- 取消事务(DISCARD),事务中的命令都不会执行
- 执行事务（exec）

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k1
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) "v1"

```



> 编译型异常（代码有问题！命令有错），事务所有的命令都不会被执行

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> set
(error) ERR wrong number of arguments for 'set' command
127.0.0.1:6379> get k3
QUEUED
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k3
(nil)

```



> 运行时异常（1/0），如果事务队列中存在语法性错误，其他命令可以正常执行，错误命令抛出异常

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> incr k1#字符串是无法增1的，运行时异常
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> get k1
QUEUED
127.0.0.1:6379> exec
1) (error) ERR value is not an integer or out of range
2) OK
3) "v2"
4) "v1"

```

**监视**

watch 实现乐观锁，类似用version实现的乐观锁，只在事务中起作用

```bash
如果事务执行失败
unwatch#先放弃监视
watch money #重新监视
```

![image-20200823121811271](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200823121811.png)

### Jedis

> Redis 官方推荐的 java 连接开发工具，使用 Java 操作Redis的中间件

导包

```xml
        <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.3.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.73</version>
        </dependency>
```

> 常用的API操作，和前面的是一样的
>
> List,Set,Hash,Zset,String

连接测试

远程连接注意更改配置文件

![image-20200823143555167](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200823143555.png)

```java
// centos7关闭firewall
//systemctl stop firewalld.service

    
    public class TestConnection {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("192.168.42.129",6379);
        System.out.println(jedis.ping());
        jedis.close();
    }
}
```

> 事务的使用

```java
    public static void main(String[] args) {
        Jedis jedis = new Jedis("192.168.42.129",6379);
        //开启事务
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("name","liang");
        jsonObject.put("age","12");
        String s = jsonObject.toJSONString();
        jedis.flushDB();
        Transaction multi = jedis.multi();
        try {
            multi.set("user1",s);
            multi.set("user2",s);
            int i=1/0;//抛出异常，事务执行失败
            multi.exec();//执行事务
        }catch (Exception e){
            multi.discard();//放弃事务
            e.printStackTrace();
        }finally {
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));
            jedis.close();
        }


    }
```

```java
//使用watch
        Jedis jedis = new Jedis("192.168.42.129",6379);
        //开启事务
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("name","liang");
        jsonObject.put("age","12");
        String s = jsonObject.toJSONString();
        jedis.flushDB();
        jedis.set("user1",s);
        jedis.watch("user1");
        try {
            TimeUnit.SECONDS.sleep(10);//其它连接在这段时间修改观测值
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Transaction multi = jedis.multi();
        try {
            multi.set("user1",s);
            multi.set("user2",s);
            multi.exec();//执行事务
        }catch (Exception e){
            multi.discard();//放弃事务
            e.printStackTrace();
        }finally {
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));
            jedis.close();
        }
```



### SpringBoot 整合

spring data redis 操作 redis

SpringBoot 2.x后，将 jedis 换成了lettuce

jedis: Jedis在实现上是直接连接的redis server，如果在多线程环境下是非线程安全的，这个时候只有使用连接池，为每个Jedis实例增加物理连接

```java
        //多线程需要使用连接池，为每个线程配置一个实际物理连接
//        JedisPool jedisPool = new JedisPool();
//        Jedis resource = jedisPool.getResource();
```



lettuce: Lettuce的连接是基于Netty的，连接实例（StatefulRedisConnection）可以在多个线程间并发访问，因为StatefulRedisConnection是线程安全的，所以一个连接实例，不需要连接池，减少资源浪费

**源码分析**

```java
public class RedisAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")//我们可以自己定义一个RedisTemplate
	public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
			throws UnknownHostException {
        //默认的RedisTemplate没有过多的设置，redis对象都需要序列化
        //两个泛型都是Object,我们使用需要转换String,Object
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean //由于String常用这里单独提出来了StringRedisTemplate
	public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory)
			throws UnknownHostException {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

}
```

导入依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```



配置连接

```yml
#SpringBoot的所有配置类，都有一个自动配置类 RedisAutoConfiguration
#自动配置类绑定一个properties配置文件RedisProperties

spring:
  redis:
    host: 192.168.42.129
    port: 6379
```



测试

```java
    @Autowired
    RedisTemplate redisTemplate;

    @Test
    void contextLoads() {

//        redisTemplate.opsForValue().set("name","liang");
//        redisTemplate.opsForGeo();
//        redisTemplate.opsForHash();
//        redisTemplate.opsForHyperLogLog();
//        redisTemplate.opsForList();
//        redisTemplate.opsForSet();
//        redisTemplate.opsForZSet();
//        redisTemplate.opsForHash();
        //=============事务
//        redisTemplate.multi();
//        redisTemplate.discard();
//        redisTemplate.exec();
//获取连接对象
//        RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
//        connection.flushAll();
//        connection.flushDb();
//        connection.shutdown();

    }
```

![image-20200823210146919](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200823210147.png)



![image-20200823210300743](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200823210300.png)

> 序列化失败

![image-20200824094039168](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824094046.png)

> User实现序列化接口后，jdk就能正常序列化了

```bash
127.0.0.1:6379> keys *
1) "\xac\xed\x00\x05t\x00\x04user"
默认jdk序列化后存储在redis中的k,v都会转义,redis中可读性极差
```

> 编写自己的redistemplate

```java
@Configuration
public class RedisConfig {
    //编写自己的RedisTemplate
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);

        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.activateDefaultTyping(LaissezFaireSubTypeValidator.instance,
                ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

        //key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        //hash的key采用String
        template.setHashKeySerializer(stringRedisSerializer);
        //value采用json
        template.setValueSerializer(jackson2JsonRedisSerializer);
        //hash的value也采用json
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        return template;
    }
}

```

> 常用API封装

```java
@Component
public final class RedisUtil {


    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    // =============================common============================

    /**
     * 指定缓存失效时间
     *
     * @param key  键
     * @param time 时间(秒)
     * @return 30
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }

    /**
     * 根据key 获取过期时间
     *
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }

    /**
     * 判断key是否存在
     *
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 删除缓存
     *
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String... key) {
        if (key != null && key.length > 0) {
            if (key.length == 1) {
                redisTemplate.delete(key[0]);
            } else {
                redisTemplate.delete(CollectionUtils.arrayToList(key));
            }
        }
    }

    // ============================String=============================

    /**
     * 普通缓存获取
     *
     * @param key 键
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }

    /**
     * 普通缓存放入
     *
     * @param key   键
     * @param value 值
     * @return true成功 false失败
     */
    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 普通缓存放入并设置时间
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 递增
     *
     * @param key   键
     * @param delta 要增加几(大于0)
     * @return
     */

    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }

    /**
     * 递减
     *
     * @param key   键
     * @param delta 要减少几(小于0)
     * @return
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }

    // ================================Map=================================

    /**
     * HashGet
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return 值
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(key, item);
    }

    /**
     * 获取hashKey对应的所有键值
     *
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * HashSet
     *
     * @param key 键
     * @param map 对应多个键值
     * @return true 成功 false 失败
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * HashSet 并设置时间
     *
     * @param key  键
     * @param map  对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @param time  时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 删除hash表中的值
     *
     * @param key  键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(key, item);
    }

    /**
     * 判断hash表中是否有该项的值
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(key, item);
    }

    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     *
     * @param key  键
     * @param item 项
     * @param by   要增加几(大于0)
     * @return
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, by);
    }

    /**
     * hash递减
     *
     * @param key  键
     * @param item 项
     * @param by   要减少记(小于0)
     * @return
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, -by);
    }

    // ============================set=============================

    /**
     * 根据key获取Set中的所有值
     *
     * @param key 键
     * @return
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key   键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将数据放入set缓存
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 将set数据放入缓存
     *
     * @param key    键
     * @param time   时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0) {
                expire(key, time);
            }
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 获取set缓存的长度
     *
     * @param key 键
     * @return
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 移除值为value的
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 移除的个数
     */
    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }

    }

    // ===============================list=================================

    /**
     * 获取list缓存的内容
     *
     * @param key   键
     * @param start 开始
     * @param end   结束 0 到 -1代表所有值
     * @return
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            e.printStackTrace();
            return null;

        }

    }

    /**
     * 获取list缓存的长度
     *
     * @param key 键
     * @return
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 通过索引 获取list中的值
     *
     * @param key   键
     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     * @return
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }

    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @return 436
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据索引修改list中的某条数据
     *
     * @param key   键
     * @param index 索引
     * @param value 值
     * @return
     */
    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 移除N个值为value
     *
     * @param key   键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */
    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
}

```

### Redis配置文件

> 单位

![image-20200824111747480](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824111747.png)

> 可以包含多个配置文件，类似java中的import

![image-20200824111920520](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824111920.png)

> 网络

```bash
bind 127.0.0.1  #绑定的ip，redis监听的ip，只有该ip主机能连接到redis，可以指定多个，或者完全注释，那么就允许所有其它ip的主机来访问

protected-mode yes #保护模式

port 6379 # 占用的端口
```

> 通用GENERAL

```bash
# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes  #以守护进程的模式运行，后台运行，默认是no

# When the server runs non daemonized, no pid file is created if none is
# specified in the configuration. When the server is daemonized, the pid file
# is used even if not specified, defaulting to "/var/run/redis.pid".
#
# Creating a pid file is best effort: if Redis is not able to create it
# nothing bad happens, the server will start and run normally.
pidfile /var/run/redis_6379.pid #以后台方式运行，需要指定一个pid文件

#日志
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably) 生产环境
# warning (only very important / critical messages are logged)
loglevel notice

logfile "" #日志输出位置，""代表标准输出
database 16 #数据库的数量，类似mysql中的表数量，默认16个
always-show-logo yes #启动是否显示logo


```

> 快照

```bash
# 如果900s内，至少有一个key修改了，就进行持久化操作
save 900 1
# 如果300s内，至少有10个key修改了，就进行持久化操作
save 300 10
# 如果60s内，至少有10000个key修改了，就进行持久化操作
save 60 10000
stop-writes-on-bgsave-error yes #持久化出错是否还继续工作
rdbcompression yes#是否压缩rdb文件，需要消耗cpu资源
rdbchecksum yes#保存rdb文件时进行校验
dir ./#rdb的保存目录
```

> REPLICATION 复制 ，主从复制需要配置



从机需要配置

![image-20200824171133446](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824171133.png)





> SECURITY 安全

默认没有密码，可以在配置文件中加密码

```bash
requirepass 123456#设置密码
```

命令设置密码

```bash
config set requirepass "123456"
#登陆
auth 123456
```

> 限制CLIENTS

```BASH
# maxclients 10000 最大客户端连接数
maxmemory <bytes> #最大容量

# maxmemory-policy noeviction #内存满的淘汰策略
以下是可以设置的模式
# volatile-lru -> Evict using approximated LRU, only keys with an expire set.
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU, only keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> Remove a random key having an expire set.
# allkeys-random -> Remove a random key, any key.
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# noeviction -> Don't evict anything, just return an error on write operations.

```

> APPEND ONLY 模式 aof配置

```bash
appendonly no  #默认不开启aof,大部分情况rdb就够用

appendfilename "appendonly.aof" #持久化名字


# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.

# appendfsync always #有修改就同步，消耗性能
appendfsync everysec #每一秒同步一次，会丢失1s内的数据
# appendfsync no #不执行同步，操作系统自己同步数据，速度快，不太安全

```

### Redis持久化（重要）



#### RDB （Redis DataBase）

在主从复制中RDB就是用来备份的，放在从机上面

> 保存过程图

![image-20200824144203655](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824144203.png)



![image-20200824144411394](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824144411.png)

> 触发生成.rdb规则

- shutdown
- flushall
- 配置文件中的save规则满足
- save命令



> 从.rdb文件恢复数据

```bash
127.0.0.1:6379> config get dir #查看.rdb需要放置的目录
1) "dir"
2) "/usr/local/bin"
# .rdb放在上述，redis启动目录中，redis自己会恢复
```

优点：

1、适合大规模数据恢复

缺点：

1、需要一定的时间间隔执行操作，意外宕机，最后的一次的修改数据就没了

2、fork进程需要占用空间



#### AOF

将所有执行的命令都存到文件中，保存，恢复的时候，再全部执行一遍

![image-20200824150435705](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824150435.png)

![image-20200824150845717](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824150845.png)

默认是不开启的

```bash
appendonly no #要开启改为yes即可
```

重启redis 生效

如果 aof 文件有问题，redis启动不起来，可以使用`redis-check-aof --fix appendonly.aof`修复

rdb有问题是没法修复的，需要最好备份

启动不起来，可以考虑删掉aof,rdb,不得已的操作

> 重写规则说明

aof 默认就是文件的无限制追加，文件会越来越大

![image-20200824153555302](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824153555.png)

触发规则：　

1、手动：

为了减小aof文件的体量，可以手动发送“bgrewriteaof”指令，通过子进程生成更小体积的aof，然后替换掉旧的、大体量的aof文件。

2、自动：

　　1）auto-aof-rewrite-percentage 100

　　2）auto-aof-rewrite-min-size 64mb

　　这两个配置项的意思是，在aof文件体量超过64mb，且比上次重写后的体量增加了100%时自动触发重写

![image-20200824154256122](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824154256.png)

​		需要注意的是，在这里子进程把数据转为写指令存入新的AOF文件时，记录的只是每个数据的最后一次写指令，也就是最新的数据，不会记录之前冗余的操作，所以这样会很大程度的缩小AOF的体量，同时，该操作是产生新的AOF文件进行写入，而不是在原有文件上的修改，通过上图也可以看出来。而缓存中叠加到新的aof的操作仍是新增的全部操作，但是这些数据已经很有限，相比之前的一股脑添加，这种机制很好的解决的AOF文件不断增大的问题。



优点：

使用 AOF 持久化会让 Redis 变得非常耐久（much more durable）：**你可以设置不同的 fsync 策略**，比如无 fsync ，每秒钟一次 fsync ，或者每次执行写入命令时 fsync 。 **AOF 的默认策略为每秒钟 fsync 一次**，在这种配置下，Redis 仍然可以保持良好的性能，并且就算发生故障停机，也**最多只会丢失一秒钟的数据**（ fsync 会在后台线程执行，所以主线程可以继续努力地处理命令请求）。

缺点：

AOF 文件的体积通常要大于 RDB 文件的体积,修复速度慢

aof 运行效率慢于rdb

**扩展**

![image-20200824155517161](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824155517.png)

### Redis 发布订阅

![image-20200824162412553](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824162412.png)

![image-20200824162431176](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824162431.png)

![image-20200824162519067](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824162519.png)

> 程序中使用

```java
//        RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
//        connection.subscribe();
```

> 原理

![image-20200824162734979](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824162735.png)

**使用场景：**

1、实现消息系统

2、实时聊天

3、订阅，关注

复杂场景使用消息中间件rabbitmq,kafka等

### Redis 主从复制

> 概念

![image-20200824163128483](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824163128.png)

![image-20200824163241914](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824163242.png)

一般一主二从，三台redis服务器

#### 环境搭建

```bash
127.0.0.1:6379> info replication #查看当前库信息
# Replication
role:master #角色为master
connected_slaves:0 #从机为0个
master_replid:268fd3c158cc24ccf62e604ecd406616b9725fe7
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

```

复制配置文件

- 修改端口
- 修改pid文件
- 修改dump.rdb文件
- 修改logfile文件名

![image-20200824170032595](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824170032.png)

==默认情况下都是master==

```bash
127.0.0.1:6380> slaveof 127.0.0.1 6379 # slaveof host port  认host:port为老大
OK
127.0.0.1:6380> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:3
master_sync_in_progress:0
slave_repl_offset:14
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:1fe70caf7bd362ea870f4b0974a7bf034fb9cc01
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:14
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:14

127.0.0.1:6379> info replication #显示主机信息
# Replication
role:master
connected_slaves:2 #有两个从
slave0:ip=127.0.0.1,port=6380,state=online,offset=70,lag=1
slave1:ip=127.0.0.1,port=6381,state=online,offset=70,lag=1
master_replid:1fe70caf7bd362ea870f4b0974a7bf034fb9cc01
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:70
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:70


```

==真是环境我们是在配置文件中配置== ，replication处配置

主机可以读写，从机只能执行读操作

主机断开后，从机依旧可读取，如果主机重新连接上来，执行写操作，从机还是可以继续读到新写的数据

如果是命令行配置，从机断开后，重新连接会变成主机，再次改为从机后，所有数据会直接从master同步过来

> 复制原理

![image-20200824172209985](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824172210.png)

> 层层链路

M -- S -- S，中间的S既当主，也当从，实际上中间的角色依旧是从，无法进行写

> 如果M宕机

手动执行`salveof no one` 可以成为主节点，其他从节点再手动配置

如果宕机的主节点回来了，就是一个孤零零的主节点

### 哨兵模式

自动选举老大的模式

![image-20200824200332230](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824200332.png)

![image-20200824200416520](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824200416.png)

![image-20200824200450646](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824200450.png)

![image-20200824200658538](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824200658.png)

> 配置文件

```bash
# Example sentinel.conf

# *** IMPORTANT ***
#
# By default Sentinel will not be reachable from interfaces different than
# localhost, either use the 'bind' directive to bind to a list of network
# interfaces, or disable protected mode with "protected-mode no" by
# adding it to this configuration file.
#
# Before doing that MAKE SURE the instance is protected from the outside
# world via firewalling or other means.
#
# For example you may use one of the following:
#
# bind 127.0.0.1 192.168.1.1
#
# protected-mode no

# port <sentinel-port>
# The port that this sentinel instance will run on
port 26379  #如果有哨兵集群就需要多个配置文件，配置不同的端口

# By default Redis Sentinel does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis-sentinel.pid when
# daemonized.
daemonize no

# When running daemonized, Redis Sentinel writes a pid file in
# /var/run/redis-sentinel.pid by default. You can specify a custom pid file
# location here.
pidfile /var/run/redis-sentinel.pid

# Specify the log file name. Also the empty string can be used to force
# Sentinel to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile ""

# sentinel announce-ip <ip>
# sentinel announce-port <port>
#
# The above two configuration directives are useful in environments where,
# because of NAT, Sentinel is reachable from outside via a non-local address.
#
# When announce-ip is provided, the Sentinel will claim the specified IP address
# in HELLO messages used to gossip its presence, instead of auto-detecting the
# local address as it usually does.
#
# Similarly when announce-port is provided and is valid and non-zero, Sentinel
# will announce the specified TCP port.
#
# The two options don't need to be used together, if only announce-ip is
# provided, the Sentinel will announce the specified IP and the server port
# as specified by the "port" option. If only announce-port is provided, the
# Sentinel will announce the auto-detected local IP and the specified port.
#
# Example:
#
# sentinel announce-ip 1.2.3.4

# dir <working-directory>
# Every long running process should have a well-defined working directory.
# For Redis Sentinel to chdir to /tmp at startup is the simplest thing
# for the process to don't interfere with administrative tasks such as
# unmounting filesystems.
dir /tmp

# sentinel monitor <master-name> <ip> <redis-port> <quorum>
#
# Tells Sentinel to monitor this master, and to consider it in O_DOWN
# (Objectively Down) state only if at least <quorum> sentinels agree. 至少quorum个哨兵认为它宕机了，就认为它处于客观宕机状态
#
# Note that whatever is the ODOWN quorum, a Sentinel will require to
# be elected by the majority of the known Sentinels in order to
# start a failover, so no failover can be performed in minority.
#
# Replicas are auto-discovered, so you don't need to specify replicas in
# any way. Sentinel itself will rewrite this configuration file adding
# the replicas using additional configuration options.
# Also note that the configuration file is rewritten when a
# replica is promoted to master.
#
# Note: master name should not include special characters or spaces.
# The valid charset is A-z 0-9 and the three characters ".-_".
sentinel monitor mymaster 127.0.0.1 6379 2

# sentinel auth-pass <master-name> <password>  #如果redis master配置了密码，这里要配置，从节点必须和主节点密码一致
#
# Set the password to use to authenticate with the master and replicas.
# Useful if there is a password set in the Redis instances to monitor.
#
# Note that the master password is also used for replicas, so it is not
# possible to set a different password in masters and replicas instances
# if you want to be able to monitor these instances with Sentinel.
#
# However you can have Redis instances without the authentication enabled
# mixed with Redis instances requiring the authentication (as long as the
# password set is the same for all the instances requiring the password) as
# the AUTH command will have no effect in Redis instances with authentication
# switched off.
#
# Example:
#
# sentinel auth-pass mymaster MySUPER--secret-0123passw0rd

# sentinel auth-user <master-name> <username>
#
# This is useful in order to authenticate to instances having ACL capabilities,
# that is, running Redis 6.0 or greater. When just auth-pass is provided the
# Sentinel instance will authenticate to Redis using the old "AUTH <pass>"
# method. When also an username is provided, it will use "AUTH <user> <pass>".
# In the Redis servers side, the ACL to provide just minimal access to
# Sentinel instances, should be configured along the following lines:
#
#     user sentinel-user >somepassword +client +subscribe +publish \
#                        +ping +info +multi +slaveof +config +client +exec on

# sentinel down-after-milliseconds <master-name> <milliseconds>
#
# Number of milliseconds the master (or any attached replica or sentinel) should
# be unreachable (as in, not acceptable reply to PING, continuously, for the
# specified period) in order to consider it in S_DOWN state (Subjectively
# Down).
#
# Default is 30 seconds.
sentinel down-after-milliseconds mymaster 30000

# requirepass <password>
#
# You can configure Sentinel itself to require a password, however when doing
# so Sentinel will try to authenticate with the same password to all the
# other Sentinels. So you need to configure all your Sentinels in a given
# group with the same "requirepass" password. Check the following documentation
# for more info: https://redis.io/topics/sentinel

# sentinel parallel-syncs <master-name> <numreplicas>
#
# How many replicas we can reconfigure to point to the new replica simultaneously
# during the failover. Use a low number if you use the replicas to serve query
# to avoid that all the replicas will be unreachable at about the same
# time while performing the synchronization with the master.
sentinel parallel-syncs mymaster 1

# sentinel failover-timeout <master-name> <milliseconds>
#
# Specifies the failover timeout in milliseconds. It is used in many ways:
#
# - The time needed to re-start a failover after a previous failover was
#   already tried against the same master by a given Sentinel, is two
#   times the failover timeout.
#
# - The time needed for a replica replicating to a wrong master according
#   to a Sentinel current configuration, to be forced to replicate
#   with the right master, is exactly the failover timeout (counting since
#   the moment a Sentinel detected the misconfiguration).
#
# - The time needed to cancel a failover that is already in progress but
#   did not produced any configuration change (SLAVEOF NO ONE yet not
#   acknowledged by the promoted replica).
#
# - The maximum time a failover in progress waits for all the replicas to be
#   reconfigured as replicas of the new master. However even after this time
#   the replicas will be reconfigured by the Sentinels anyway, but not with
#   the exact parallel-syncs progression as specified.
#
# Default is 3 minutes.
sentinel failover-timeout mymaster 180000

# SCRIPTS EXECUTION
#
# sentinel notification-script and sentinel reconfig-script are used in order
# to configure scripts that are called to notify the system administrator
# or to reconfigure clients after a failover. The scripts are executed
# with the following rules for error handling:
#
# If script exits with "1" the execution is retried later (up to a maximum
# number of times currently set to 10).
#
# If script exits with "2" (or an higher value) the script execution is
# not retried.
#
# If script terminates because it receives a signal the behavior is the same
# as exit code 1.
#
# A script has a maximum running time of 60 seconds. After this limit is
# reached the script is terminated with a SIGKILL and the execution retried.

# NOTIFICATION SCRIPT
#
# sentinel notification-script <master-name> <script-path>
# 
# Call the specified notification script for any sentinel event that is
# generated in the WARNING level (for instance -sdown, -odown, and so forth).
# This script should notify the system administrator via email, SMS, or any
# other messaging system, that there is something wrong with the monitored
# Redis systems.
#
# The script is called with just two arguments: the first is the event type
# and the second the event description.
#
# The script must exist and be executable in order for sentinel to start if
# this option is provided.
#
# Example:
#
# sentinel notification-script mymaster /var/redis/notify.sh

# CLIENTS RECONFIGURATION SCRIPT
#
# sentinel client-reconfig-script <master-name> <script-path>
#
# When the master changed because of a failover a script can be called in
# order to perform application-specific tasks to notify the clients that the
# configuration has changed and the master is at a different address.
# 
# The following arguments are passed to the script:
#
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
#
# <state> is currently always "failover"
# <role> is either "leader" or "observer"
# 
# The arguments from-ip, from-port, to-ip, to-port are used to communicate
# the old address of the master and the new address of the elected replica
# (now a master).
#
# This script should be resistant to multiple invocations.
#
# Example:
#
# sentinel client-reconfig-script mymaster /var/redis/reconfig.sh

# SECURITY
#
# By default SENTINEL SET will not be able to change the notification-script
# and client-reconfig-script at runtime. This avoids a trivial security issue
# where clients can set the script to anything and trigger a failover in order
# to get the program executed.

sentinel deny-scripts-reconfig yes

# REDIS COMMANDS RENAMING
#
# Sometimes the Redis server has certain commands, that are needed for Sentinel
# to work correctly, renamed to unguessable strings. This is often the case
# of CONFIG and SLAVEOF in the context of providers that provide Redis as
# a service, and don't want the customers to reconfigure the instances outside
# of the administration console.
#
# In such case it is possible to tell Sentinel to use different command names
# instead of the normal ones. For example if the master "mymaster", and the
# associated replicas, have "CONFIG" all renamed to "GUESSME", I could use:
#
# SENTINEL rename-command mymaster CONFIG GUESSME
#
# After such configuration is set, every time Sentinel would use CONFIG it will
# use GUESSME instead. Note that there is no actual need to respect the command
# case, so writing "config guessme" is the same in the example above.
#
# SENTINEL SET can also be used in order to perform this configuration at runtime.
#
# In order to set a command back to its original name (undo the renaming), it
# is possible to just rename a command to itsef:
#
# SENTINEL rename-command mymaster CONFIG CONFIG

```



==注意==

```bash
#启动哨兵
redis-sentinel redisconfig/sentinel.conf

#测试时
配置文件如下配置一行就行
sentinel monitor mymaster <ip> <redis-port> 1 #注意ip必须是master的redis地址，不能写localhost,程序需要这个ip来访问redis,需要是程序能发现的ip
```

> 当宕机的主机回来时，只能做新主机的从机

![image-20200824201357356](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200824201357.png)

### Redis 缓存穿透与雪崩

目前简单知道解决方案

**缓存穿透**

用户查询一个数据，发现key不在缓存中，去持久层数据库中查询。多次使用不存在的key去请求，就会导致缓存穿透

> 解决方案

![image-20200825141510335](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200825141517.png)

![image-20200825141626401](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200825141626.png)

![image-20200825141638404](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200825141638.png)

**缓存击穿**（量太大，缓存过期）

![image-20200825141818730](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200825141818.png)

> 解决方案

![image-20200825141924697](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200825141924.png)



**缓存雪崩**

![image-20200825142610985](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200825142611.png)

![image-20200825142623405](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200825142623.png)

> 解决方案

![image-20200825142716731](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200825142716.png)





### 一些小命令

```bash
monitor# 进入监测模式，可以实时看到发送的请求，执行的命令
```

### 小结

服务器应该：

> 3V------海量Velume
>
> ​       多样Variety
>
> ​       实时Velocity
>
> 3高-----高并发
>
> ​       高可扩
>
> ​       高性能 

​       多样Variety

​       实时Velocity

3高-----高并发

​       高可扩

​       高性能 

主从复制 （配置哨兵）：

```yaml
spring:
  redis:
    sentinel:
      master: mymaster
      nodes: 192.168.42.129:26379 #哨兵如果是集群就配置多个地址即可
```



读写分离 ：

主从复制配置好，如果只使用原来的连接，只会在master上进行读写，需要额外的配置，专门配置读的连接，生成对应的template

集群（cluster）:

多个主结点，多个从节点，去中心化，需要redis配置文件配置，springboot配置文件也要配置

### 参考资料

[视频资料-狂神说](https://www.bilibili.com/video/av840034966?p=1)