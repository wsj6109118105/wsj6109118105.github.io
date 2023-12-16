---
title: Redis基础
date: 2021-08-13 18:54:41
tags: Redis
---
## redis启动
```bash
redis-server wconfig/redis.conf   # 开启服务
redis-cli -p 6379   #连接，默认6379端口

127.0.0.1:6379> shutdown   #关闭
not connected> exit
```
## 基本命令
```bash
127.0.0.1:6379> select 2   # 切换数据库  默认有16个数据库
OK

127.0.0.1:6379[2]> dbsize    #查看db大小
(integer) 0

127.0.0.1:6379> set name wanglufei  #添加数据
OK

127.0.0.1:6379> get name    # 查看数据
"wanglufei"

127.0.0.1:6379> keys *      # 查看所有的key
1) "name"

127.0.0.1:6379[2]> keys *    
1) "name"

127.0.0.1:6379[2]> flushdb    # 清除当前数据库
OK

127.0.0.1:6379[2]> keys *
(empty array)

127.0.0.1:6379[2]> flushall     # 清除全部数据库  

127.0.0.1:6379> exists name   # 判断键是否存在
(integer) 1

127.0.0.1:6379> move name 1    # 移除key
(integer) 1

127.0.0.1:6379> EXPIRE name 20  #设置键的过期时间
(integer) 1

127.0.0.1:6379> ttl name  #查看剩余时间
(integer) 11

127.0.0.1:6379> type age  #查看key的类型
string

```
> redis是单线程的

redis是基于内存操作的，CPU不是redis的性能瓶颈，redis的瓶颈是根据机器的内存和网络带宽，既然可以用单线程，就使用单线程

**redis为什么单线程还这么快**
- 误区1：高性能的服务器一定是多线程的
- 误区2：多线程（CPU调度，单CPU会上下文切换）一定比单线程效率高！
- 核心：redis是将所有的数据全部放在内存中的，所以说使用单线程去操作是效率最高的，多线程（CPU上下文会切换：耗时的操作！），对于内存系统来说，如果没有上下文切换效率就是最高的！
## 正式开始
> 介绍

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作**数据库**、**缓存**和**消息中间件**。 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。
## String
```bash
127.0.0.1:6379> APPEND age kkk   #追加字符串，如果当前key不存在，就相当于setkey
(integer) 5

127.0.0.1:6379> STRLEN age         #  获取字符串长度
(integer) 5

##############################################################
127.0.0.1:6379> set views 0
OK

127.0.0.1:6379> get views
"0"

127.0.0.1:6379> incr views    # 增加
(integer) 1

127.0.0.1:6379> incr views
(integer) 2

127.0.0.1:6379> get views
"2"

127.0.0.1:6379> decr views     # 减少
(integer) 1

127.0.0.1:6379> get views
"1"

127.0.0.1:6379> incrby views 10   # 增加一个步长
(integer) 11

127.0.0.1:6379> DECRBY views 10   # 减少一个步长
(integer) 1

#############################################################
#字符串范围  range
127.0.0.1:6379> set kay1 wanglufei  
OK

127.0.0.1:6379> get kay1
"wanglufei"

127.0.0.1:6379> GETRANGE kay1 0 5   # 截取字符串，下标从0开始
"wanglu"

127.0.0.1:6379> GETRANGE kay1 0 -1  # 获取全部字符串
"wanglufei"

# 替换字符串
127.0.0.1:6379> set kay2 abcde
OK

127.0.0.1:6379> get kay2
"abcde"

127.0.0.1:6379> SETRANGE kay2 1 xx  #指定替换开始的位置为1，后面替换为xx
(integer) 5

127.0.0.1:6379> get kay2   #看到将1之后长度为2的字符穿替换为xx
"axxde"
############################################################
# setex(set with expire) 设置过期时间
# setnx(set if not exit) 如果不存在再设置  在分布式锁中常用

127.0.0.1:6379> setex kay3 60 sss  #设置过期时间60s
OK

127.0.0.1:6379> ttl kay3   # 查看剩余时间
(integer) 55

127.0.0.1:6379> setnx mykey redis   #  如果不存在设置
(integer) 1

127.0.0.1:6379> get mykey  # 查看
"redis"

127.0.0.1:6379> setnx mykey mongodb  # 如果不存在设置
(integer) 0                     #  说明设置失败，已经存在

127.0.0.1:6379> get mykey     #查看
"redis"
#############################################################
#mset  批量设置值
#mget  批量获取值
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3   #设置
OK

127.0.0.1:6379> keys *
1) "k1"
2) "k3"
3) "k2"

127.0.0.1:6379> mget k1 k2 k3   #获取
1) "v1"
2) "v2"
3) "v3"

127.0.0.1:6379> msetnx k1 v1 k4 v4  #不存在则设置
(integer) 0            # 结果失败

127.0.0.1:6379> get k4   # key4不存在，说明上面的操作是个原子性操作，要么同时成功，要么同时失败
(nil)

127.0.0.1:6379> mset user:1:name zhangsan user:1:age 99  #设置对象方式
OK

127.0.0.1:6379> mget user:1:name user:1:age
1) "zhangsan"
2) "99"
#############################################################
# getset 先get再set
127.0.0.1:6379> getset db redis
(nil)

127.0.0.1:6379> get db
"redis"

127.0.0.1:6379> getset db mongodb
"redis"

127.0.0.1:6379> get db
"mongodb"

```
String类型的使用场景：
- 计数器
- 统计多单位数量
- 对象缓存存储

## List
list所有命令都是l开头
```bash
127.0.0.1:6379> LPUSH list one two three
(integer) 3

127.0.0.1:6379> LRANGE list 0 -1   #list是倒着存的（从左往右）
1) "three"
2) "two"
3) "one"

127.0.0.1:6379> RPUSH list four   #从右往做插入
(integer) 4

127.0.0.1:6379> LRANGE list 0 -1   #可见four插入到了one右侧
1) "three"
2) "two"
3) "one"
4) "four"

127.0.0.1:6379> LPOP list 1   #从左边移除
1) "three"

127.0.0.1:6379> RPOP list    #从右边移除
"four"

127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"

127.0.0.1:6379> LINDEX list 0   #获取对应下标的值
"two"

127.0.0.1:6379> LINDEX list 1
"one"

127.0.0.1:6379> LLEN list   #返回列表长度
(integer) 3

127.0.0.1:6379> lpush list three   #再添加一个three
(integer) 4

127.0.0.1:6379> LREM list 2 three   #移除两个three
(integer) 2

127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
#############################################################
127.0.0.1:6379> RPUSH mylist hello hello1 hello2 hello3
(integer) 4

127.0.0.1:6379> LRANGE mylist 0 -1
1) "hello"
2) "hello1"
3) "hello2"
4) "hello3"

127.0.0.1:6379> LTRIM mylist 1 2  #通过下标截取list,从1到2
OK

127.0.0.1:6379> LRANGE mylist 0 -1
1) "hello1"
2) "hello2"
#############################################################
# rpoplpush 移除列表最后一个元素，并添加到一个新的列表
127.0.0.1:6379> LPUSH mylist hello hello1 hello2
(integer) 3
127.0.0.1:6379> RPOPLPUSH mylist list
"hello"

127.0.0.1:6379> LRANGE mylist 0 -1
1) "hello2"
2) "hello1"

127.0.0.1:6379> LRANGE list 0 -1
1) "hello"
#############################################################
#  lset  将列表指定下标的值替换为另一个值，如果不存在会报错
127.0.0.1:6379> LPUSH list value1
(integer) 1

127.0.0.1:6379> LRANGE list 0 0
1) "value1"

127.0.0.1:6379> lset list 0 item
OK

127.0.0.1:6379> LRANGE list 0 0
1) "item"
#############################################################
#linsert  将一个value插入到列表中
127.0.0.1:6379> RPUSH mylist hello
(integer) 1

127.0.0.1:6379> RPUSH mylist world
(integer) 2

127.0.0.1:6379> LRANGE mylist 0 -1
1) "hello"
2) "world"

127.0.0.1:6379> LINSERT mylist before world value  #可以选择插入在前面还是后面
(integer) 3

127.0.0.1:6379> LRANGE mylist 0 -1
1) "hello"
2) "value"
3) "world"

```
>小结

- 实际上是一个链表，before Node after, left,right 都可以插入值
- 如果key不存在，创建新的链表
- 如果key存在，新增内容
- 如果移除所有值，空链表，也表示不存在
- 在两边插入或改动值，效率最高！

消息队列（lpush rpop）
栈（lpush lpop）
## set
set中的值不能重复
```bash
127.0.0.1:6379> sadd myset hello #添加值
(integer) 1

127.0.0.1:6379> sadd myset wanglufei
(integer) 1

127.0.0.1:6379> sadd myset lufei
(integer) 1

127.0.0.1:6379> sadd myset mengqi
(integer) 1

127.0.0.1:6379> smembers myset  #查看
1) "wanglufei"
2) "hello"
3) "lufei"
4) "mengqi"

127.0.0.1:6379> SISMEMBER myset lufei  #查看是否存在
(integer) 1                          #存在

127.0.0.1:6379> SISMEMBER myset suolong   
(integer) 0                        #不存在
#############################################################
127.0.0.1:6379> SCARD myset     #查看元素个数
(integer) 4

127.0.0.1:6379> SREM myset hello      #移除元素
(integer) 1

127.0.0.1:6379> SMEMBERS myset
1) "wanglufei"
2) "lufei"
3) "mengqi"
#############################################################
set无序不重复元素，抽随机
127.0.0.1:6379> SRANDMEMBER myset  #随机抽出一个元素
"lufei"

127.0.0.1:6379> SRANDMEMBER myset
"mengqi"
#############################################################
删除指定的key,删除随机的key
127.0.0.1:6379> spop myset   #随机删除一个元素
"wanglufei"
127.0.0.1:6379> SPOP myset
"lufei"
#############################################################
将一个指定的值移到另外一个集合中
127.0.0.1:6379> sadd myset hello
(integer) 1
127.0.0.1:6379> sadd myset world
(integer) 1
127.0.0.1:6379> sadd myset wanglufei
(integer) 1
127.0.0.1:6379> sadd myset1 lufei
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "wanglufei"
2) "hello"
3) "world"
127.0.0.1:6379> SMEMBERS myset1
1) "lufei"
127.0.0.1:6379> SMOVE myset myset1 hello  #移动hello
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "wanglufei"
2) "world"
127.0.0.1:6379> SMEMBERS myset1
1) "hello"
2) "lufei"
#############################################################
差集   SDIFF
交集   SINTER
并集   SUNION
127.0.0.1:6379> sadd key1 a
(integer) 1
127.0.0.1:6379> sadd key1 b
(integer) 1
127.0.0.1:6379> sadd key1 c
(integer) 1
127.0.0.1:6379> sadd key2 c
(integer) 1
127.0.0.1:6379> sadd key2 d
(integer) 1
127.0.0.1:6379> sadd key2 e
(integer) 1
127.0.0.1:6379> SDIFF key1 key2   #差集
1) "b"
2) "a"
127.0.0.1:6379> SINTER key1 key2   #交集
1) "c"
127.0.0.1:6379> SUNION key1 key2   #并集
1) "a"
2) "c"
3) "b"
4) "e"
5) "d"
```
## hash
map集合，key-map;这时候值为一个map
```bash
127.0.0.1:6379> hset myhash filed1 wanglufei  #set一个key-value
(integer) 1
127.0.0.1:6379> hget myhash filed1    #查看一个
"wanglufei"
127.0.0.1:6379> hset myhash filed2 lufei filed3 suolong  #set多个个key-value
(integer) 2
127.0.0.1:6379> HMGET myhash filed1 filed2 filed3   #一次查看多个
1) "wanglufei"
2) "lufei"
3) "suolong"
127.0.0.1:6379> HGETALL myhash   #查看全部
1) "filed1"
2) "wanglufei"
3) "filed2"
4) "lufei"
5) "filed3"
6) "suolong"
127.0.0.1:6379> HDEL myhash filed1   #删除指定key的字段
(integer) 1
127.0.0.1:6379> HGETALL myhash
1) "filed2"
2) "lufei"
3) "filed3"
4) "suolong"
#############################################################
127.0.0.1:6379> HLEN myhash   #查看hash的长度
(integer) 2

127.0.0.1:6379> HEXISTS myhash field2  #查看指定字段是否存在
(integer) 0
127.0.0.1:6379> HEXISTS myhash filed2
(integer) 1
#############################################################
# 获得所有的key   hkeys
# 获得所有的value   HVALS
127.0.0.1:6379> hkeys myhash   #获得所有的key
1) "filed2"
2) "filed3"

127.0.0.1:6379> HVALS myhash  #获得所有的value
1) "lufei"
2) "suolong"

127.0.0.1:6379> hset myhash field5 5
(integer) 1
127.0.0.1:6379> HINCRBY myhash field 1
(integer) 1
127.0.0.1:6379> HINCRBY myhash field5 1
(integer) 6
127.0.0.1:6379> HSETNX myhash field 5
(integer) 0
```
>小结：可以看出hash更加适合用来作对象的存储，而String适合存储字符串
## Zset(有序集合)
在set的基础上增加了一个值：sadd k1 v1,zadd k1 score1 v1
```bash
127.0.0.1:6379> zadd mylist 1 one #添加一个值
(integer) 1
127.0.0.1:6379> zadd mylist 2 two
(integer) 1
127.0.0.1:6379> zadd mylist 3 three
(integer) 1
127.0.0.1:6379> ZRANGE mylist 0 -1   #查看所有
1) "one"
2) "two"
3) "three"
#############################################################
排序如何实现
127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf #升序，但是无法降序排列
1) "qiaoba"
2) "xiaoming"
3) "lufei"

127.0.0.1:6379> ZREVRANGE salary 0 -1    #降序排列
1) "lufei"
2) "qiaoba"

127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf withscores #带上数值
1) "qiaoba"
2) "50"
3) "xiaoming"
4) "2500"
5) "lufei"
6) "10000"
#############################################################
移除元素：
127.0.0.1:6379> zrange salary 0 -1
1) "qiaoba"
2) "xiaoming"
3) "lufei"
127.0.0.1:6379> zrem salary xiaoming   #执行移除
(integer) 1
127.0.0.1:6379> zrange salary 0 -1
1) "qiaoba"
2) "lufei"

127.0.0.1:6379> zcard salary    #查看元素个数
(integer) 2
#############################################################
127.0.0.1:6379> ZCOUNT salary 20 100    #统计位于区间中的值
(integer) 1        #乔巴  50  位于区间
127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf withscores
1) "qiaoba"
2) "50"
3) "lufei"
4) "10000"
```
> 其他命令可查看官方文档：http://www.redis.cn/commands.html

## geospatial
> geoadd 将指定的地理空间位置（纬度、经度、名称）添加到指定的key中。
```bash
# 规则：两极无法直接添加，参数 key 值（经度、纬度、名称）
127.0.0.1:6379> geoadd china:city 116.40 39.90 beijing
(integer) 1
127.0.0.1:6379> geoadd china:city 121.47 31.23 shanghai
(integer) 1
127.0.0.1:6379> geoadd china:city 112.85 35.50 jincheng
(integer) 1
127.0.0.1:6379> geoadd china:city 120.15 30.28 hangzhou
(integer) 1
127.0.0.1:6379> geoadd china:city 115.48 38.86 baodin
(integer) 1
#错误
127.0.0.1:6379> geoadd china:city 39.90 116.40 beijing
(error) ERR invalid longitude,latitude pair 39.900000,116.400000

```
>geopos  从key里返回所有给定位置元素的位置（经度和纬度）。
```bash
127.0.0.1:6379> GEOPOS china:city beijing
1) 1) "116.39999896287918091"
   2) "39.90000009167092543"
127.0.0.1:6379> GEOPOS china:city beijing jincheng
1) 1) "116.39999896287918091"
   2) "39.90000009167092543"
2) 1) "112.84999877214431763"
   2) "35.50000044366486662"

```
> geodist  返回两个给定位置之间的距离  

指定单位的参数 unit 必须是以下单位的其中一个：
- m 表示单位为米。
- km 表示单位为千米。
- mi 表示单位为英里。
- ft 表示单位为英尺。
```bash
127.0.0.1:6379> GEODIST china:city beijing shanghai   #北京到上海的直线距离
"1067378.7564"
127.0.0.1:6379> GEODIST china:city beijing shanghai km  #单位km
"1067.3788"
127.0.0.1:6379> GEODIST china:city beijing jincheng   #北京到晋城的直线距离
"580488.6194"
127.0.0.1:6379> GEODIST china:city beijing jincheng km  #单位km
"580.4886"

```
> georadius :以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。

范围可以使用以下其中一个单位：
- m 表示单位为米。
- km 表示单位为千米。
- mi 表示单位为英里。
- ft 表示单位为英尺。  

附近的人(获得所有附近的人的定位)通过半径查询
```bash
127.0.0.1:6379> GEORADIUS china:city 110 30 1000 km
1) "hangzhou"
2) "jincheng"
127.0.0.1:6379> GEORADIUS china:city 110 30 1500 km
1) "hangzhou"
2) "shanghai"
3) "jincheng"
4) "baodin"
5) "beijing"
127.0.0.1:6379> GEORADIUS china:city 110 30 1500 km withdist  #携带距离返回
1) 1) "hangzhou"
   2) "976.4868"
2) 1) "shanghai"
   2) "1105.9098"
3) 1) "jincheng"
   2) "667.2204"
4) 1) "baodin"
   2) "1105.7190"
5) 1) "beijing"
   2) "1245.2858"
127.0.0.1:6379> GEORADIUS china:city 110 30 1500 km withcoord  # 携带经度纬度返回
1) 1) "hangzhou"
   2) 1) "120.15000075101852417"
      2) "30.2800007575645509"
2) 1) "shanghai"
   2) 1) "121.47000163793563843"
      2) "31.22999903975783553"
3) 1) "jincheng"
   2) 1) "112.84999877214431763"
      2) "35.50000044366486662"
4) 1) "baodin"
   2) 1) "115.48000127077102661"
      2) "38.85999893055183207"
5) 1) "beijing"
   2) 1) "116.39999896287918091"
      2) "39.90000009167092543"

```
> GEORADIUSBYMEMBER:可以找出位于指定范围内的元素,中心点是由给定的位置元素决定的.
```bash
127.0.0.1:6379> GEORADIUSBYMEMBER china:city beijing 1500 km withdist withcoord
1) 1) "hangzhou"
   2) "1122.7998"
   3) 1) "120.15000075101852417"
      2) "30.2800007575645509"
2) 1) "shanghai"
   2) "1067.3788"
   3) 1) "121.47000163793563843"
      2) "31.22999903975783553"
3) 1) "jincheng"
   2) "580.4886"
   3) 1) "112.84999877214431763"
      2) "35.50000044366486662"
4) 1) "baodin"
   2) "140.1294"
   3) 1) "115.48000127077102661"
      2) "38.85999893055183207"
5) 1) "beijing"
   2) "0.0000"
   3) 1) "116.39999896287918091"
      2) "39.90000009167092543"
```
> geohash:返回一个或多个位置元素的 Geohash 表示

```bash
127.0.0.1:6379> GEOHASH china:city beijing jincheng
1) "wx4fbxxfke0"
2) "ww21zywf7x0"
```
> geo 底层实现其实是使用zset,可以使用zset命令操作geo
```bash
127.0.0.1:6379> ZRANGE china:city 0 -1  #使用zset查看
1) "hangzhou"
2) "shanghai"
3) "jincheng"
4) "baodin"
5) "beijing"
127.0.0.1:6379> ZREM china:city beijing  #使用zset移除
(integer) 1
127.0.0.1:6379> ZRANGE china:city 0 -1   
1) "hangzhou"
2) "shanghai"
3) "jincheng"
4) "baodin"
```
## Hyperloglog
可以用来做基数统计，消耗的内存是一定的
```bash
127.0.0.1:6379> pfadd mykey a b c d e f g h i j k   #插入数据
(integer) 1
127.0.0.1:6379> PFCOUNT mykey    #统计数量
(integer) 11
127.0.0.1:6379> pfadd mykey1 l m n o p q r s t u v w x y z  
(integer) 1
127.0.0.1:6379> PFCOUNT mykey1
(integer) 15
127.0.0.1:6379> PFMERGE mykey2 mykey mykey1     #合并
OK
127.0.0.1:6379> PFCOUNT mykey2    #结果26个英文字母显示只有25个，我也没有搞明白是为什么
(integer) 25

```
> 可以拿来统计网站访问人数等
## Bitmap
> 位存储(只有两个状态的都可以使用Bitmaps)
```bash
#用mykey来记录打卡，记录6天，1打卡，0未打卡
127.0.0.1:6379> SETBIT mykey 0 1
(integer) 0
127.0.0.1:6379> SETBIT mykey 1 1
(integer) 0
127.0.0.1:6379> SETBIT mykey 2 0
(integer) 0
127.0.0.1:6379> SETBIT mykey 3 0
(integer) 0
127.0.0.1:6379> SETBIT mykey 4 1
(integer) 0
127.0.0.1:6379> SETBIT mykey 5 0
(integer) 0
127.0.0.1:6379> GETBIT mykey 3    # 查看某天是否打卡
(integer) 0
127.0.0.1:6379> GETBIT mykey 1
(integer) 1
127.0.0.1:6379> BITCOUNT mykey 0 5  # 统计
(integer) 3

```
## 事务处理
redis事务的本质：一组命令的集合，一个事务中的所有命令都会被序列话，在事务执行的过程中，会按照顺序执行！
一次性，顺序性，排他性。

**redis事务没有隔离级别的概念**

所有的命令在事务中，并没有直接被执行！只有发起执行的时候才执行！

**redis单条命令是保证原子性的，单事务是不保证原子性的**

redis的事务：
- 开启事务(multi)
- 命令入队(....)
- 执行事务(exec)

> 正常执行事务！
```bash
127.0.0.1:6379> multi #开启事务
OK
#命令入队
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> get k3
QUEUED
127.0.0.1:6379(TX)> exec #执行
1) OK
2) OK
3) OK
4) "v3"

```
>放弃事务
```bash
127.0.0.1:6379> multi  #开启事务
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k4 v4
QUEUED
127.0.0.1:6379(TX)> DISCARD  #放弃事务，命令不会执行
OK
127.0.0.1:6379> get k4
(nil)

```
>编译型异常(代码又问题！命令出差)，事务中的所有命令都不会被执行！

>运行时异常,如果事务队列中存在语法性错误，执行命令的时候，其他命令可以正常执行

>监视(watch)
```bash
127.0.0.1:6379> set monty 100
OK
127.0.0.1:6379> set cost 0
OK
127.0.0.1:6379> WATCH monty  #监视
OK
127.0.0.1:6379> multi  #事务正常结束，数据期间没有发生变动，正常执行成功
OK
127.0.0.1:6379(TX)> DECRBY monty 30
QUEUED
127.0.0.1:6379(TX)> INCRBY cost 30
QUEUED
127.0.0.1:6379(TX)> exec
1) (integer) 70
2) (integer) 30
```
测试多线程修改值，使用watch可以当作redis的乐观锁操作
```bash
127.0.0.1:6379> WATCH monty   #监视
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> DECRBY monty 30
QUEUED
127.0.0.1:6379(TX)> INCRBY monty 30
QUEUED
127.0.0.1:6379(TX)> exec #执行之前，另外一个线程修改了值，会让事务执行失败
(nil)
```
## redis.conf
1.配置文件 unit单位对大小写不敏感
2.包含
3.网络
```bash
bind 127.0.0.1 -::1   #绑定的IP
protected-mode yes    #保护模式
port 6379       #端口
```
4.通用
```bash
daemonize yes   #以守护进程方式运行

pidfile /var/run/redis_6379.pid   #如果以后台方式运行就需要指定一个pid文件

#日志
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice

logfile ""  #日志文件名


databases 16  #数据库数量默认16

always-show-logo no  #是否显示标志
```
5.快照，持久化，在规定的时间内，执行了多少次操作，则会持久化大豆文件 .rdb.aof
redis是内存数据库，如果没有持久话，那么数据断电即失
```bash
#  持久化规则
# 如果3600秒内，至少有一个key进行了修改，我们进行持久化操作
# save 3600 1
# 如果300秒内，至少10个key进行了修改，我们进行持久化操作
# save 300 100
# 如果60秒内，至少10000个key进行了修改，我们进行持久化操作
# save 60 10000

stop-writes-on-bgsave-error yes  #持久化出错，还是否进行工作


rdbcompression yes #是否压缩rdb文件，需要消耗一些cpu资源！

rdbchecksum yes  #保存rdb文件时，进行错误检查校验！

dir ./   #rdb保存目录
```
5.复制
```bash
```



6.安全  
可以在这里设置密码，默认是没有密码


7.客户端限制
```bash
# maxclients 10000  #默认最大连接数
#maxmemory <bytes>  #最大内存的配置
# maxmemory-policy noeviction   #内存达到上限的处理策略
```

8.APPEND ONLY MODE  aof配置
```bash
appendonly no # 默认不开启aof模式，默认使用rdb方式
appendfilename "appendonly.aof"   #持久化文件名字
appendfsync everysec  #每秒执行一次 sync，可能会丢失这一秒的数据
```
## 持久化
> rdb
1.什么是持久化


**触发机制**
- sava的规则满足的情况下，会自动触发rdb规则
- 执行flushall命令，也会触发我们的rdb规则！
- 退出redis,也会产生rdb文件

备份会自动生成一个dump.rdb文件

**如何恢复rdb文件**  
只需要将rdb文件放在我们redis启动目录就可以，redis启动的时候会自动检查，恢复其中的数据

> aof
将我们的所有命令(读的操作不记录)都记录下来，恢复的时候把命令全部执行一遍

如果aof文件有错误，则redis启动不了，需要修复aof文件redis提供了`redis-check-aof --fix`来修复

**优点和缺点**
- 优点：每一次修改都同步，文件的完整性会更号；没秒同步，可能会丢失一秒数据；从不同步，效率最高
- 缺点：aof远远大于rdb,修复速度比rdb慢！aof运行效率也要比rdb慢！

参考视频链接：https://www.bilibili.com/video/BV1S54y1R7SB


