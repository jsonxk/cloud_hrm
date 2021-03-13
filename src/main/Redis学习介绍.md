# Redis学习随记

### Nosql概述

​       Nosql=not only sql 泛指非关系型数据库。

传统的关系数据库很难对付web2.0时代，主要是大量高并发对于mysql等关系数据库的不友好。

##### NoSQL的四大分类：

###### 1.kv键值对

Redis    

Tair 

Memcache 

###### 2.文档类型的数据库（bson格式和json一样）

MongoDB(一般必须要掌握)

​    MongoDB是一个基于分布式文件存储的数据库，c++编写，主要用来处理大量的文档！

​    MongoDB是一个介于关系型数据库和非关系型数据库之间的产品，是NoSQL数据库中功能最丰富的数据库。

ConthDB是一个国外的数据库

###### 3.列存储数据库

HBase

分布式文件数据库

###### 4.图形关系数据库

图形关系数据库并不是存储图片的数据库 而是存储关系的数据库。

Neo4j和infoGrid;

![几种非关系数据库对比](C:\Users\91479\AppData\Roaming\Typora\typora-user-images\image-20201028072358163.png)

# Redis的数据类型

## 5大基本数据类型

#### String

## String类型

###### Append：拼接字符串

```bash
127.0.0.1:6379> set k1 hello
OK
127.0.0.1:6379> append k1 redis  ##拼接redis到k1后面
(integer) 10
127.0.0.1:6379> get k1
"helloredis"
127.0.0.1:6379>
```

###### Strlen：返回字符串长度

```bash
127.0.0.1:6379> strlen k1  ##返回k1的长度
(integer) 10
```

###### Incr: 将字符串加1.

```bash
127.0.0.1:6379> incr k1  ##不是数值型字符 incr函数报错
(error) ERR value is not an integer or out of range
127.0.0.1:6379> set k1 1
OK
127.0.0.1:6379> incr k1  ##数值型字符串 incr成功+1
(integer) 2
127.0.0.1:6379> get k1
"2"
127.0.0.1:6379>
```

###### Decr: 将字符串减1.

```bash
127.0.0.1:6379> get k1
"2"
127.0.0.1:6379> decr k1  ##数值型字符串成功+1
(integer) 1
127.0.0.1:6379> get k1
"1"
127.0.0.1:6379>
```

###### Incrby: 将字符串加固定场长度  例如 incrBy key 10 （自增10）

```bash
127.0.0.1:6379> get k1
"1"
127.0.0.1:6379> incrby k1 20  ##相当于incr 10次
(integer) 21
127.0.0.1:6379> get k1
"21"
```

###### DecrBy: 将字符串减少固定长度 例如 decrBy key 10

```bash
127.0.0.1:6379> decrby k1 50  ##减少50 相当于decr 50次
(integer) -29
127.0.0.1:6379> get k1
"-29"
```

###### Getrange key start end： 获取字符串从start开始到end结束 例如 getrange key 2 9（截取key为‘key’的字符串值的2到9为字符）  查看全部字符串 getrange key 0 -1（获取全部字符串）

```bash
127.0.0.1:6379> set str helloRedis
OK
127.0.0.1:6379> getrange str 5 9  ##获取str第6到10位字符组成字符串 本身操作不会改变原有字符串
"Redis"
127.0.0.1:6379> getrange str 0 -1  ##获取str所有字符
"helloRedis"
127.0.0.1:6379> get str
"helloRedis"
```

###### SetRange：替换指定位置的字符串 setRange key offindex “value” （将key为‘key’的字符串值 从offindex开始替换为‘value’）

```bash
127.0.0.1:6379> setrange str 5 Mysql  ##替换字符串 6位后面字符为 ‘Mysql’  会改变字符串本身
(integer) 10
127.0.0.1:6379> get str
"helloMysql"
```

###### setex（Set with expire）：#设置过期时间和值

```bash
setex mykey seconds value=SET mykey value + EXPIRE mykey seconds ##setex 等价于 set +expire
127.0.0.1:6379> get str
"hello"
127.0.0.1:6379> setex str 20 hi ##将str的值设置为 hi 过期时间为20s
OK
127.0.0.1:6379> get str
"hi"
127.0.0.1:6379> get str
(nil)
```

###### setnx（set if not exist）：#不存在再设置（分布式锁中常用）

```bash
###如果该键存在，则不会对该键的值做任何操作 如果不存在则创建该键值对。

127.0.0.1:6379> setnx mykey hello  ##设置不存在的mykey的值为 hello
(integer) 1
127.0.0.1:6379> setnx str 1
(integer) 1
127.0.0.1:6379> setnx mykey hi ##再次设置不存在的mykey的值为 hi 但实际上已近存在
(integer) 0						##修改结果为0，说明操作失败了
127.0.0.1:6379> get mykey    
"hello"                      ##证实操作失败了
```

###### mset和mget:批量设置键值对。例如mset k1 v1 k2 v2 k3 v3(同时设置了三个值)

```bash
127.0.0.1:6379> mset key1 v1 key2 v2 key3 v3  ##批量设置k的值
OK
127.0.0.1:6379> mget key1 key2 key3   ##批量获取k的值
1) "v1"
2) "v2"
3) "v3"
```

###### msetnx k1 v1 k2 v2  #msetnx是一个原子性操作，要么一起成功要么一起失败 就是说只要有一个存在就失败

```bash
127.0.0.1:6379> msetnx mystr1 hello mystr2 hi mystr v3  ##都不存在一起成功
(integer) 1
127.0.0.1:6379> mget mystr1 mystr2 mystr  
1) "hello"
2) "hi"
3) "v3"
127.0.0.1:6379> msetnx mystr hello mystr4 v5 ##有一个存在（mystr） 操作失败
(integer) 0  ##操作失败
127.0.0.1:6379> get mystr4    ##获取不到key mystr4的值
(nil)
```



#### List

   **所有的list操作几乎都是以‘l’开头的**

###### Lpush list value1 :将value1左插入到list中

```bash
127.0.0.1:6379> lpush list1 v1 v2   ##向list1头（左）顺序插入 v1 v2
(integer) 2
127.0.0.1:6379> lrange list1 0 -1  ##查看list1的结果
1) "v2"
2) "v1"
```

###### Rpush list value2：将value2右插入到list中

```bash
127.0.0.1:6379> Rpush list1  v3 v4 ## 队尾插入 v3 v4
(integer) 4

127.0.0.1:6379> lrange list1 0 -1
1) "v2"
2) "v1"
3) "v3"
4) "v4"
```

###### Lrange list index1 index2 获取队列的index1到index2的值

```bash
127.0.0.1:6379> lrange list1 0 -1  # 查看队列中的所有值
1) "v2"
2) "v1"
3) "v3"
4) "v4"
```

###### Lpop和Rpop list从左或右移除值

```bash
127.0.0.1:6379> lrange list1 0 -1
1) "v2"
2) "v1"
3) "v3"
4) "v4" 
127.0.0.1:6379> lpop list1   ##头（左）弹出一个值
"v2"
127.0.0.1:6379> lrange list1 0 -1
1) "v1"
2) "v3"
3) "v4"
127.0.0.1:6379> rpop list1  ##尾（右）弹出一个值
"v4"
127.0.0.1:6379> lrange list1 0 -1
1) "v1"
2) "v3"
```

###### Lindex key index 获取下标为index的值 

```bash
127.0.0.1:6379> lindex list1 0  ##获取队列下标为0的值
"v1"
```

###### Llen list 获取列表的长度值

```bash
127.0.0.1:6379> lpush list1 v5 v6 v7
(integer) 5
127.0.0.1:6379> llen list1  ##获取表长度
(integer) 5
```

###### Lrem list n value ；移除指定value的n个值

```bash
127.0.0.1:6379> lrem list1 2 v5   ##移除list1中2个值为v5的值
(integer) 1  ###由于只有一个v5的值，所以结果返回1
127.0.0.1:6379> lrange list1 0 -1
1) "v7"
2) "v6"
3) "v1"
4) "v3"
```

###### Ltrim list index1 index2 修剪（截取）list的index1到index2的值

```bash
127.0.0.1:6379> lrange list1 0 -1
1) "v7"
2) "v6"
3) "v1"
4) "v3"
127.0.0.1:6379> ltrim list1 0 2  ##裁剪list1
OK
127.0.0.1:6379> lrange list1 0 -1
1) "v7"
2) "v6"
3) "v1"
```

###### RpopLpush list1 list2 将list1中的最后元素移到list2中

```bash
127.0.0.1:6379> lrange list1 0 -1
1) "v7"
2) "v6"
3) "v1"
127.0.0.1:6379> lpush list2 1 2 3 4 5
(integer) 5
127.0.0.1:6379> RpopLpush list1 list2  ##将list1尾部元素弹出到list2头
"v1"
127.0.0.1:6379> lrange list1 0 -1
1) "v7"
2) "v6"
127.0.0.1:6379> lrange list2 0 -1
1) "v1"
2) "5"
3) "4"
4) "3"
5) "2"
6) "1"
```

###### Lset list index value ：如果list存在且下标index存在则替换该list的index下标的value值为‘value’

```bash
127.0.0.1:6379> lset list1 3 hello  ##下标不存在 报错
(error) ERR index out of range
127.0.0.1:6379> lset list1 1 hi   ##存在则替换为新值
OK
127.0.0.1:6379> lrange list1 0 -1
1) "v7"
2) "hi"
```

###### Linsert list before/after value value2 向list中值为value的前/后插入值为value的值

```bash
127.0.0.1:6379> lrange list1 0 -1
1) "v7"
2) "hi"
127.0.0.1:6379> linsert list1 before v7 hello  ##在list1中值为v7的前面添加hello值
(integer) 3
127.0.0.1:6379> lrange list1 0 -1
1) "hello"
2) "v7"
3) "hi"
```

#### Set

#### Hash

###### Hashes,由field和关联的value组成的map。field和value都是字符串的

```bash
hset key field1 v1 field2 v2  
hmset key field1 v1 field2 v2  ##两者基本同价
hgetall key  ##获取key下所有field和值
hvals key  ##获取key下所有field的值，不包含field
hget key field ##获取key下field对应的值
hmget key field1 field2 ... ##批量获取key下field对应的值
hlen key  ##获取key下fields的数量
例如：
127.0.0.1:6379> hset people:zhangsan name zhansan age 12  ##hset使用
(integer) 2
127.0.0.1:6379> hgetall people:zhangsan  ##查看k下所有field和值
1) "name"
2) "zhansan"
3) "age"
4) "12"
127.0.0.1:6379> hmset people:zhangsan phone 13655671322 address beijing  ##hmset使用
OK
127.0.0.1:6379> hgetall people:zhangsan
1) "name"
2) "zhansan"
3) "age"
4) "12"
5) "phone"
6) "13655671322"
7) "address"
8) "beijing"
127.0.0.1:6379> hget people:zhangsan name  ##获取field为name的值
"zhansan"
127.0.0.1:6379> hmget people:zhangsan name phone address  ##批量获取指定field的值
1) "zhansan"
2) "13655671322"
3) "beijing"
127.0.0.1:6379> hlen people:zhangsan  ##获取key为people：张三的hash表中field的数量
(integer) 4
127.0.0.1:6379> hvals people:zhangsan  ###获取key下field的所有值 ，仅仅包含值
1) "zhansan"
2) "14"
3) "13655671322"
4) "beijing"
5) "122235.19999999999708962"
```

###### hincrby 只能增加整数型字符field的值

```bash
hincrby（整数型）和hincrbyfloat（数值型）
127.0.0.1:6379> hincrby people:zhangsan age 2
(integer) 14
127.0.0.1:6379> hget people:zhangsan age
"14"
127.0.0.1:6379> hset people:zhangsan money 122233.00
(integer) 1
127.0.0.1:6379> hincrby people:zhangsan money 21  ###浮点型 hincrby不能增加
(error) ERR hash value is not an integer
127.0.0.1:6379> hincrbyfloat people:zhangsan money 2.2  ###浮点型 hincrbyfloat可以增加
"122235.19999999999708962"
127.0.0.1:6379>
```

#### Zset

### 三种特殊数据类型

#### Geospatial 地理位置

##### 1.GeoAdd 添加地理位置

```bash
#geoadd key 经度 纬度 别名
geoadd china:city 116.40 39.90 beijing  #设置别名为北京key为china：city的经纬度
```

##### 2.GeoPos  获取当前定位信息

```bash
#geopos key 别名
geopos china：city beijing #获取key为china：city别名为北京的经纬度信息
```

##### 3.Geodist 获取两个key 别名之间的距离

```bash
#geodist key 别名1 别名2
#单位 m km mi(英里) ft(英尺)
geodist china:city beijing beijing1
命令完整表达式：GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]
参数解释# m ：米，默认单位。
# km ：千米。
# mi ：英里。
# ft ：英尺。
#WITHDIST: 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回。
#WITHCOORD: 将位置元素的经度和维度也一并返回。
#WITHHASH: 以 52 位有符号整数的形式， 返回位置元素经过原始 geohash 编码的有序集合分值。 这个选项主要用#于底层应用或者调试， 实际中的作用并不大。
#COUNT 限定返回的记录数。
#ASC: 查找结果根据距离从近到远排序。
#DESC: 查找结果根据从远到近排序。
```

##### 4.GeoRadius 以经纬度为中心，返回key中包含在指定半径的所有别名元素

```bash
命令完整表达式：GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]
参数解释# m ：米，默认单位。
# km ：千米。
# mi ：英里。
# ft ：英尺。
#WITHDIST: 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回。
#WITHCOORD: 将位置元素的经度和维度也一并返回。
#WITHHASH: 以 52 位有符号整数的形式， 返回位置元素经过原始 geohash 编码的有序集合分值。 这个选项主要用#于底层应用或者调试， 实际中的作用并不大。
#COUNT 限定返回的记录数。
#ASC: 查找结果根据距离从近到远排序。
#DESC: 查找结果根据从远到近排序。
例子：georadius china:city 116 44 500 km withcoord withdist count 1 asc
```

##### 5.GeoRadiusMember

```bash
#georadiusbymember 和 GEORADIUS 命令一样， 都可以找出位于指定范围内的元素， 但是 georadiusbymember 的中心点是由给定的位置元素决定的， 而不是使用经度和纬度来决定中心点。
```

##### 6.Geohash

```bash
geohash key member1 member2 返回一个或多个元素位置的geohash表示(标准geohash值)
例子：127.0.0.1:6379[1]> geohash china:city beijing beijing2
1) "wx4fbxxfke0"
2) "wx6g88xukx0"
127.0.0.1:6379[1]>

```

##### 7.Geo底层实现是基于基本数据类型Zset的，所以可以用zset命令来操作Geo元素

```bash
例如：zrem来删除geo元素
127.0.0.1:6379[1]> zrange china:city 0 -1
1) "beijing"
2) "beijing1"
3) "beijing2"
127.0.0.1:6379[1]> zrem china:city beijing2  //删除key为china：city beijing2的geo元素
(integer) 1
127.0.0.1:6379[1]> zrange china:city 0 -1
1) "beijing"
2) "beijing1"
127.0.0.1:6379[1]>
```



#### Hyperloglog

Redis的基数统计算法（统计不同元素的数量，有一定的容错）  本来java提供了set集合和其强大的api，但是由于redis的hyperloglog基数统计算法消耗的内存远远小于java set，官方介绍是2的64次方个元素只消耗12kb内存。当然在降低内存的同时他 不会存储元素本身，所以不适用于需要取元素本身的值的场合。

```bash
127.0.0.1:6379[1]> pfadd mykey a d f g h j k l n h ###添加统计集合元素
(integer) 1
127.0.0.1:6379[1]> pfcount mykey ###查看元素基数(集合元素的不同数量)
(integer) 9
127.0.0.1:6379[1]> pfadd mykey2 a d f h n m g z x
(integer) 1
127.0.0.1:6379[1]> pfcount mykey2
(integer) 9
127.0.0.1:6379[1]> pfmerge mymergekey mykey mykey2 ####合并2个集合生成新集合(去掉重复集合)
OK
127.0.0.1:6379[1]> pfcount mymergekey
(integer) 12
127.0.0.1:6379[1]>
```



#### Bitmap 位存储

bitmaps位图 主要用于存储2种状态的数据（0，1）

```bash
127.0.0.1:6379> setbit iflogin 0 1   ##设置key的值第一位位1，‘真’
(integer) 0
127.0.0.1:6379> setbit iflogin 1 1
(integer) 0
127.0.0.1:6379> setbit iflogin 2 1
(integer) 0
127.0.0.1:6379> bitcount iflogin    ##统计key的值中为真的数量
(integer) 3
```



### Redis事务

redis事务和mysql事务并不完全相同。mysql要求acid都满足，而redis不需要。

###### 		一、原子性（atomicity)

一个事务要么全部提交成功，要么全部失败回滚，不能只执行其中的一部分操作，这就是事务的原子性。

###### 		二、一致性（consistency)

事务的执行不能破坏数据库数据的完整性和一致性，一个事务在执行之前和执行之后，数据库都必须处于一致性状态。

​		如果数据库系统在运行过程中发生故障，有些事务尚未完成就被迫中断，这些未完成的事务对数据库所作的修改有一部分已写入物理数据库，这是数据库就处于一种不正确的状态，也就是不一致的状态

###### 		三、隔离性（isolation）

​		事务的隔离性是指在并发环境中，并发的事务时相互隔离的，一个事务的执行不能不被其他事务干扰。不同的事务并发操作相同的数据时，每个事务都有各自完成的数据空间，即一个事务内部的操作及使用的数据对其他并发事务时隔离的，并发执行的各个事务之间不能相互干扰。

###### 		四、持久性（durability）

一旦事务提交，那么它对数据库中的对应数据的状态的变更就会永久保存到数据库中。--即使发生系统崩溃或机器宕机等故障，只要数据库能够重新启动，那么一定能够将其恢复到事务成功结束的状态

##### **Redis事务的概念：**

​		Redis 事务的本质是一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

　　总结说：redis事务就是一次性的、顺序性的、排他性的执行一个队列中的一系列命令。

**Redis事务没有隔离级别的概念：**

​		批量操作在发送 EXEC 命令前被放入队列缓存，并不会被实际执行，也就不存在事务内的查询要看到事务里的更新，事务外查询不能看到。

**Redis不保证原子性：**

​	Redis中，单条命令是原子性执行的，但事务不保证原子性，且没有回滚。事务中任意命令执行失败，其余的命令仍会被执行。

**Redis事务的三个阶段：**

- 开始事务（multi）
- 命令入队 
- 执行事务 （exec）

##### Redis事务相关命令

- watch :类似mysql的乐观锁，只做监视key的value有没有在事务multi后exce前被其他事务改变，若发生改变则会打断该事务执行。

- ```bash
  watch key ##在开启事务前进行key监控
  ```

- multi ：标志着一个事务的开始

- exec：事务执行 （一旦执行，之前加的监控锁都会被取消掉）

- discard：取消事务 放弃事务块（队列）中的所有事务，同时也会放弃监控（接触乐观锁）

- unwatch: 取消对事物中所有key的监视

##### Redis事务特点

​	（1）编译型的错误：redis事务在exec时如果出现命令本身的语法错误，即类似java编译时异常，那么事务在exec时，所有事务都无法执行。

​	（2）运行是的错误：redis事务在exec时若没有语法错误，但是执行时出错，那么正确的事务队列中的命令还是会被执行，执行出错的命令会报错。

##### Jedis

​	Jedis是Redis官方推荐的Java连接开发工具。要在Java开发中使用好Redis中间件。

​    依赖：

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.6.0</version> 
</dependency>
```

使用方式：

```java
public void connectionTest() {
jedis = new Jedis("127.0.0.1", 6379);//redis的地址以及连接端口
//jedis.auth("helloworld");  //开启密码验证（配置文件中为 requirepass helloworld）的时候需要
    //执行该方法
       jedis.get(if(jedis.exists("foo"))) //获取key
       jedis.set("key","value")   //设置key
       jedis.expire("key", 60);// 设置60秒后该key过期
       jedis.persist("key"); //移除key的过期时间

       byte[] bytes = jedis.dump("key");// 导出key的值


       Set<String> set = jedis.keys("*") //查询匹配的key  *匹配所有 ？匹配单个字符 [abc]匹配括号内部分字符
}
```



### Redis主从复制

##### 概述

​		主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master)，后者称为从节点(slave)；数据的复制是单向的，只能由主节点到从节点。

​		默认情况下，每台Redis服务器都是主节点；且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。

##### 优点

1. 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
2. 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
3. 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
4. 高可用基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

##### 主从复制建立

1. **主从复制的开启，完全是在从节点发起的；不需要我们在主节点做任何事情。**

2. 从节点开启主从复制，有3种方式：

   （1）配置文件

   在从服务器的配置文件中加入：slaveof <masterip> <masterport>

   （2）启动命令

   redis-server启动命令后加入 --slaveof <masterip> <masterport>

   （3）客户端命令

   Redis服务器启动后，直接通过客户端执行命令：slaveof <masterip> <masterport>，则该Redis实例成为从节点

3. ##### 断开复制

   ​	通过slaveof <masterip> <masterport>命令建立主从复制关系以后，可以通过slaveof no one断开。需要注意的是，从节点断开复制后，不会删除已有的数据，只是不再接受主节点新的数据变化。



#### 主从复制实现原理

![主从复制](C:\Users\91479\AppData\Roaming\Typora\typora-user-images\image-20201026073229662.png)

​		主从复制过程大体可以分为3个阶段：连接建立阶段（即准备阶段）、数据同步阶段、命令传播阶段；下面分别进行介绍。

1. ##### 连接建立和维护阶段

   ###### 步骤1：保存主节点信息

     	从节点服务器内部维护了两个字段，即masterhost和masterport字段，用于存储主节点的ip和port信息。

   需要注意的是，**slaveof是异步命令，从节点完成主节点ip和port的保存后，向发送slaveof命令的客户端直接返回OK**，实际的复制操作在这之后才开始进行。这个过程中，可以看到从节点打印日志如下：

   ###### 步骤2：建立socket连接

   ​		从节点每秒1次调用复制定时函数replicationCron()，如果发现了有主节点可以连接，便会根据主节点的ip和port，创建socket连接。如果连接成功，则：

   ​		从节点：为该socket建立一个专门处理复制工作的文件事件处理器，负责后续的复制工作，如接收RDB文件、接收命令传播等。

   ​		主节点：接收到从节点的socket连接后（即accept之后），为该socket创建相应的客户端状态，**并将从节点看做是连接到主节点的一个客户端，后面的步骤会以从节点向主节点发送命令请求的形式来进行。**

   ###### 步骤3：发送ping命令

   ​	从节点成为主节点的客户端之后，发送ping命令进行首次请求，目的是：检查socket连接是否可用，以及主节点当前是否能够处理请求。

   从节点发送ping命令后，可能出现3种情况：

   （1）返回pong：说明socket连接正常，且主节点当前可以处理请求，复制过程继续。

   （2）超时：一定时间后从节点仍未收到主节点的回复，说明socket连接不可用，则从节点断开socket连接，并重连。

   （3）返回pong以外的结果：如果主节点返回其他结果，如正在处理超时运行的脚本，说明主节点当前无法处理命令，则从节点断开socket连接，并重连。

   ###### 步骤4：身份验证

   ​	如果从节点中设置了masterauth选项，则从节点需要向主节点进行身份验证；没有设置该选项，则不需要验证。从节点进行身份验证是通过向主节点发送auth命令进行的，auth命令的参数即为配置文件中的masterauth的值。

   ​	如果主节点设置密码的状态，与从节点masterauth的状态一致（一致是指都存在，且密码相同，或者都不存在），则身份验证通过，复制过程继续；如果不一致，则从节点断开socket连接，并重连。

2. ##### 数据同步

   ​			主从节点之间的连接建立以后，便可以开始进行数据同步，该阶段可以理解为从节点数据的初始化。具体执行的方式是：从节点向主节点发送psync命令（Redis2.8以前是sync命令），开始同步。

   ​			数据同步阶段是主从复制最核心的阶段，根据主从节点当前状态的不同，可以分为全量复制和部分复制，下面会有一章专门讲解这两种复制方式以及psync命令的执行过程，这里不再详述。

   ​			需要注意的是，在数据同步阶段之前，从节点是主节点的客户端，主节点不是从节点的客户端；而到了这一阶段及以后，主从节点互为客户端。原因在于：在此之前，主节点只需要响应从节点的请求即可，不需要主动发请求，而在数据同步阶段和后面的命令传播阶段，主节点需要主动向从节点发送请求（如推送缓冲区中的写命令），才能完成复制。

   

3. ##### 命令传播阶段

   ​		数据同步阶段完成后，主从节点进入命令传播阶段；在这个阶段主节点将自己执行的写命令发送给从节点，从节点接收命令并执行，从而保证主从节点数据的一致性。

   ​		在命令传播阶段，除了发送写命令，主从节点还维持着心跳机制：PING和REPLCONF ACK。由于心跳机制的原理涉及部分复制，因此将在介绍了部分复制的相关内容后单独介绍该心跳机制。
   
   ​	**延迟与不一致**
   
   ​		需要注意的是，命令传播是异步的过程，即主节点发送写命令后并不会等待从节点的回复；因此实际上主从节点之间很难保持实时的一致性，延迟在所难免。数据不一致的程度，与主从节点之间的网络状况、主节点写命令的执行频率、以及主节点中的repl-disable-tcp-nodelay配置等有关。
   
   ​		repl-disable-tcp-nodelay no：该配置作用于命令传播阶段，控制主节点是否禁止与从节点的TCP_NODELAY；默认no，即不禁止TCP_NODELAY。当设置为yes时，TCP会对包进行合并从而减少带宽，但是发送的频率会降低，从节点数据延迟增加，一致性变差；具体发送频率与Linux内核的配置有关，默认配置为40ms。当设置为no时，TCP会立马将主节点的数据发送给从节点，带宽增加但延迟变小。
   
   一般来说，只有当应用对Redis数据不一致的容忍度较高，且主从节点之间网络状况不好时，才会设置为yes；多数情况使用默认值no。

#### 全量复制与部分复制（数据同步模式）

​		在Redis2.8以前，从节点向主节点发送sync命令请求同步数据，此时的同步方式是全量复制；在Redis2.8及以后，从节点可以发送psync命令请求同步数据，此时根据主从节点当前状态的不同，同步方式可能是全量复制或部分复制。后文介绍以Redis2.8及以后版本为例。

1. **全量复制**：用于初次复制或其他无法进行部分复制的情况，将主节点中的所有数据都发送给从节点，是一个非常重型的操作。

2. **部分复制**：用于网络中断等情况后的复制，只将中断期间主节点执行的写命令发送给从节点，与全量复制相比更加高效。需要注意的是，如果网络中断时间过长，导致主节点没有能够完整地保存中断期间执行的写命令，则无法进行部分复制，仍使用全量复制。

   ##### 全量复制过程

   Redis通过psync命令进行全量复制的过程如下：

   （1）从节点判断无法进行部分复制，向主节点发送全量复制的请求；或从节点发送部分复制的请求，但主节点判断无法进行部分复制；具体判断过程需要在讲述了部分复制原理后再介绍。

   （2）主节点收到全量复制的命令后，执行bgsave，在后台生成RDB文件，并使用一个缓冲区（称为复制缓冲区）记录从现在开始执行的所有写命令

   （3）主节点的bgsave执行完成后，将RDB文件发送给从节点；**从节点首先清除自己的旧数据，然后载入接收的****RDB文件，将数据库状态更新至主节点执行bgsave时的数据库状态

   （4）主节点将前述复制缓冲区中的所有写命令发送给从节点，从节点执行这些写命令，将数据库状态更新至主节点的最新状态

   （5）如果从节点开启了AOF，则会触发bgrewriteaof的执行，从而保证AOF文件更新至主节点的最新状态

   ###### 通过全量复制的过程可以看出

   ​	（1）主节点通过bgsave命令fork子进程进行RDB持久化，该过程是非常消耗CPU、内存(页表复制)、硬盘IO的；

   ​	（2）主节点通过网络将RDB文件发送给从节点，对主从节点的带宽都会带来很大的消耗

   ​	（3）从节点清空老数据、载入新RDB文件的过程是阻塞的，无法响应客户端的命令；如果从节点执行bgrewriteaof，也会带来额外的消耗

   ##### 部分复制过程

   ​		由于全量复制在主节点数据量较大时效率太低，因此Redis2.8开始提供部分复制，用于处理网络中断时的数据同步。

   ​		部分复制的实现，依赖于三个重要的概念：

   （1）**复制偏移量**（offset）:主从节点都维护一个offset，代表的是**主节点向从节点传递的字节数**，当进行数据同步时，主从节点的（offset）都进行同步数据字节个数的增长。

    （2）**复制积压缓冲区（先进先出队列）**：复制积压缓冲区是由主节点维护的、固定长度的、先进先出(FIFO)队列，默认大小1MB；复制积压缓冲区存储了写命令和每个字节对应的复制偏移量（offset）。

   ​	由于该缓冲区长度固定且有限，因此可以备份的写命令也有限，当主从节点offset的差距过大超过缓冲区长度时，将无法执行部分复制，只能执行全量复制。反过来说，为了提高网络中断时部分复制执行的概率，可以根据需要增大复制积压缓冲区的大小(通过配置repl-backlog-size)；例如如果网络中断的平均时间是60s，而主节点平均每秒产生的写命令(特定协议格式)所占的字节数为100KB，则复制积压缓冲区的平均需求为6MB，保险起见，可以设置为12MB，来保证绝大多数断线情况都可以使用部分复制。

   从节点将offset发送给主节点后，主节点根据offset和缓冲区大小决定能否执行部分复制：

   - ###### 如果offset偏移量之后的数据，仍然都在复制积压缓冲区里，则执行部分复制；

   - ###### 如果offset偏移量之后的数据已不在复制积压缓冲区中（数据已被挤出），则执行全量复制。

   ##### （3）服务器运行ID(runid)：

   ​		每个Redis节点(无论主从)，在启动时都会自动生成一个随机ID(每次启动都不一样)，由40个随机的十六进制字符组成；runid用来唯一识别一个Redis节点。通过info Server命令，可以查看节点的runid：

   ​		主从节点初次复制时，主节点将自己的runid发送给从节点，从节点将这个runid保存起来；当断线重连时，从节点会将这个runid发送给主节点；主节点根据runid判断能否进行部分复制：

   - 如果从节点保存的runid与主节点现在的runid相同，说明主从节点之前同步过，主节点会继续尝试使用部分复制(到底能不能部分复制还要看offset和复制积压缓冲区的情况)；
   - 如果从节点保存的runid与主节点现在的runid不同，说明从节点在断线前同步的Redis节点并不是当前的主节点，只能进行全量复制。

   

   ## psync命令的执行

   ​	在了解了复制偏移量、复制积压缓冲区、节点运行id之后，本节将介绍psync命令的参数和返回值，从而说明psync命令执行过程中，主从节点是如何确定使用全量复制还是部分复制的。

   ![psync命令详解](C:\Users\91479\AppData\Roaming\Typora\typora-user-images\image-20201027102218937.png)

   

（1）首先，从节点根据当前状态，决定如何调用psync命令：

- 如果从节点之前未执行过slaveof或最近执行了slaveof no one，则从节点发送命令为psync ? -1，向主节点请求全量复制；
- 如果从节点之前执行了slaveof，则发送命令为psync <runid> <offset>，其中runid为上次复制的主节点的runid，offset为上次复制截止时从节点保存的复制偏移量。

（2）主节点根据收到的psync命令，及当前服务器状态，决定执行全量复制还是部分复制：

- 如果主节点版本低于Redis2.8，则返回-ERR回复，此时从节点重新发送sync命令执行全量复制；
- 如果主节点版本够新，且runid与从节点发送的runid相同，且从节点发送的offset之后的数据在复制积压缓冲区中都存在，则回复+CONTINUE，表示将进行部分复制，从节点等待主节点发送其缺少的数据即可；
- 如果主节点版本够新，但是runid与从节点发送的runid不同，或从节点发送的offset之后的数据已不在复制积压缓冲区中(在队列中被挤出了)，则回复+FULLRESYNC <runid> <offset>，表示要进行全量复制，其中runid表示主节点当前的runid，offset表示主节点当前的offset，从节点保存这两个值，以备使用。

#### 主从复制存在的问题及解决方案

##### 	 1.读写分离及其中的问题

###### （1）延迟与不一致问题

​		前面已经讲到，由于主从复制的命令传播是异步的，延迟与数据的不一致不可避免。如果应用对数据不一致的接受程度程度较低，可能的优化措施包括：优化主从节点之间的网络环境（如在同机房部署）；监控主从节点延迟（通过offset）判断，如果从节点延迟过大，通知应用不再通过该从节点读取数据；使用集群同时扩展写负载和读负载等。

​		在命令传播阶段以外的其他情况下，从节点的数据不一致可能更加严重，例如连接在数据同步阶段，或从节点失去与主节点的连接时等。从节点的slave-serve-stale-data参数便与此有关：它控制这种情况下从节点的表现；如果为yes（默认值），则从节点仍能够响应客户端的命令，如果为no，则从节点只能响应info、slaveof等少数命令。该参数的设置与应用对数据一致性的要求有关；如果对数据一致性要求很高，则应设置为no。

###### （2）数据过期问题

在单机版Redis中，存在两种删除策略：

- 惰性删除：服务器不会主动删除数据，只有当客户端查询某个数据时，服务器判断该数据是否过期，如果过期则删除。

- 定期删除：服务器执行定时任务删除过期数据，但是考虑到内存和CPU的折中（删除会释放内存，但是频繁的删除操作对CPU不友好），该删除的频率和执行时间都受到了限制。

  ​	在主从复制场景下，为了主从节点的数据一致性，从节点不会主动删除数据，而是由主节点控制从节点中过期数据的删除。由于主节点的惰性删除和定期删除策略，都不能保证主节点及时对过期数据执行删除操作，因此，当客户端通过Redis从节点读取数据时，很容易读取到已经过期的数据。

  ​	Redis 3.2中，从节点在读取数据时，增加了对数据是否过期的判断：如果该数据已过期，则不返回给客户端；将Redis升级到3.2可以解决数据过期问题。

###### （3）故障切换问题

 		在没有使用哨兵的读写分离场景下，应用针对读和写分别连接不同的Redis节点；当主节点或从节点出现问题而发生更改时，需要及时修改应用程序读写Redis数据的连接；连接的切换可以手动进行，或者自己写监控程序进行切换，但前者响应慢、容易出错，后者实现复杂，成本都不算低。



###### （4）数据同步阶段问题

​		在主从节点进行全量复制bgsave时，主节点需要首先fork子进程将当前数据保存到RDB文件中，然后再将RDB文件通过网络传输到从节点。如果RDB文件过大，主节点在fork子进程+保存RDB文件时耗时过多，可能会导致从节点长时间收不到数据而触发超时；此时从节点会重连主节点，然后再次全量复制，再次超时，再次重连……这是个悲伤的循环。为了避免这种情况的发生，除了注意Redis单机数据量不要过大，另一方面就是适当增大repl-timeout值，具体的大小可以根据bgsave耗时来调整。

###### （5）复制中断问题

​		前面曾提到过，在全量复制阶段，主节点会将执行的写命令放到复制缓冲区中，该缓冲区存放的数据包括了以下几个时间段内主节点执行的写命令：bgsave生成RDB文件、RDB文件由主节点发往从节点、从节点清空老数据并载入RDB文件中的数据。当主节点数据量较大，或者主从节点之间网络延迟较大时，可能导致该缓冲区的大小超过了限制，此时主节点会断开与从节点之间的连接；这种情况可能引起全量复制->复制缓冲区溢出导致连接中断->重连->全量复制->复制缓冲区溢出导致连接中断……的循环。

​		复制缓冲区的大小由client-output-buffer-limit slave {hard limit} {soft limit} {soft seconds}配置，默认值为client-output-buffer-limit slave 256MB 64MB 60，其含义是：如果buffer大于256MB，或者连续60s大于64MB，则主节点会断开与该从节点的连接。该参数是可以通过config set命令动态配置的（即不重启Redis也可以生效）。

​		**需要注意的是，复制缓冲区是客户端输出缓冲区的一种，主节点会为每一个从节点分别分配复制缓冲区；而复制积压缓冲区则是一个主节点只有一个，无论它有多少个从节点。**



### 各场景下复制的选择及优化技巧

#### （1）第一次建立复制

​			此时全量复制不可避免，但仍有几点需要注意：如果主节点的数据量较大，应该尽量避开流量的高峰期，避免造成阻塞；如果有多个从节点需要建立对主节点的复制，可以考虑将几个从节点错开，避免主节点带宽占用过大。此外，如果从节点过多，也可以调整主从复制的拓扑结构，由一主多从结构变为树状结构（中间的节点既是其主节点的从节点，也是其从节点的主节点）；但使用树状结构应该谨慎：虽然主节点的直接从节点减少，降低了主节点的负担，但是多层从节点的延迟增大，数据一致性变差；且结构复杂，维护相当困难。

#### （2）主节点重启

主节点重启可以分为两种情况来讨论，一种是故障导致宕机，另一种则是有计划的重启。

**主节点宕机**

主节点宕机重启后，runid会发生变化，因此不能进行部分复制，只能全量复制。

实际上在主节点宕机的情况下，应进行故障转移处理，将其中的一个从节点升级为主节点，其他从节点从新的主节点进行复制；且故障转移应尽量的自动化，后面文章将要介绍的哨兵便可以进行自动的故障转移。

**安全重启：debug reload**

​		在一些场景下，可能希望对主节点进行重启，例如主节点内存碎片率过高，或者希望调整一些只能在启动时调整的参数。如果使用普通的手段重启主节点，会使得runid发生变化，可能导致不必要的全量复制。

​		为了解决这个问题，Redis提供了debug reload的重启方式：**重启后，主节点的**runid和offset都不受影响，避免了全量复制。

​		但debug reload是一柄双刃剑：它会清空当前内存中的数据，重新从RDB文件中加载，这个过程会导致主节点的阻塞，因此也需要谨慎。



####  复制相关的配置

这一节总结一下与复制有关的配置，说明这些配置的作用、起作用的阶段，以及配置方法等；通过了解这些配置，一方面加深对Redis复制的了解，另一方面掌握这些配置的方法，可以优化Redis的使用，少走坑。

配置大致可以分为主节点相关配置、从节点相关配置以及与主从节点都有关的配置，下面分别说明。

##### （1）与主从节点都有关的配置

首先介绍最特殊的配置，它决定了该节点是主节点还是从节点：

1)  slaveof <masterip> <masterport>：Redis启动时起作用；作用是建立复制关系，开启了该配置的Redis服务器在启动后成为从节点。该注释默认注释掉，即Redis服务器默认都是主节点。

2)  repl-timeout 60：与各个阶段主从节点连接超时判断有关，见前面的介绍。

##### （2）主节点相关配置

1)  repl-diskless-sync no：作用于全量复制阶段，控制主节点是否使用diskless复制（无盘复制）。所谓diskless复制，是指在全量复制时，主节点不再先把数据写入RDB文件，而是直接写入slave的socket中，整个过程中不涉及硬盘；diskless复制在磁盘IO很慢而网速很快时更有优势。需要注意的是，截至Redis3.0，diskless复制处于实验阶段，默认是关闭的。

2)  repl-diskless-sync-delay 5：该配置作用于全量复制阶段，当主节点使用diskless复制时，该配置决定主节点向从节点发送之前停顿的时间，单位是秒；只有当diskless复制打开时有效，默认5s。之所以设置停顿时间，是基于以下两个考虑：(1)向slave的socket的传输一旦开始，新连接的slave只能等待当前数据传输结束，才能开始新的数据传输 (2)多个从节点有较大的概率在短时间内建立主从复制。

3)  client-output-buffer-limit slave 256MB 64MB 60：与全量复制阶段主节点的缓冲区大小有关

4)  repl-disable-tcp-nodelay no：与命令传播阶段的延迟有关

5)  masterauth <master-password>：与连接建立阶段的身份验证有关

6)  repl-ping-slave-period 10：与命令传播阶段主从节点的超时判断有关

7)  repl-backlog-size 1mb：复制积压缓冲区的大小

8)  repl-backlog-ttl 3600：当主节点没有从节点时，复制积压缓冲区保留的时间，这样当断开的从节点重新连进来时，可以进行部分复制；默认3600s。如果设置为0，则永远不会释放复制积压缓冲区。

9)  min-slaves-to-write 3与min-slaves-max-lag 10：规定了主节点的最小从节点数目，及对应的最大延迟

##### （3）从节点相关配置

1)  slave-serve-stale-data yes：与从节点数据陈旧时是否响应客户端命令有关

2)  slave-read-only yes：从节点是否只读；默认是只读的。由于从节点开启写操作容易导致主从节点的数据不一致，因此该配置尽量不要修改。



#### Redis配置文件介绍

###### 1.单位和大小写

```bash
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes   ##单位 

# include .\path\to\local.conf
# include c:\path\to\other.conf    ##包含其他配置文件
```



###### 2.NETWORK（网络配置）

```bash
# Examples:
#
# bind 192.168.1.100 10.0.0.1    ##例子
# bind 127.0.0.1 ::1 
bind 127.0.0.1  ##指定服务器ip地址
protected-mode yes ##保护模式
port 6379 ##端口号 默认6379
```



###### 3.通用配置（general）

```bash
deamonize yes #以守护进程方式运行redis window配置文件没有 默认是no，最好开启（置为yes）

pidfile /var/run/redis_6379.pid #若以后台方式运行，那么就要指定一个pid进程文件。

# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing) 开发和测试
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably) 生产上的
# warning (only very important / critical messages are logged) 重要关键信息日志
loglevel notice    ##日志级别 notice（生产级别）

logfile ''   ##日志文件位置

database 16  ##数据库数量配置

always show logo  yes  #默认显示redis的logo

 
```



###### 4.快照 SNAPSHOTTING （redis持久化时使用）

持久化，在规定时间内，执行了多少次操作，会持久化到rdb文件或者aof文件（默认生成rdb文件） 

redis是内存数据库，断电即失，所以要持久化      

```bash
##配置持久化规则
save 900 1     ##900s 至少有1个key进行修改  那么redis会进行持久化
save 300 10    ##300s 有10个key进行修改  那么redis会进行持久化
save 60 10000  ##60s 有10000个key进行修改  那么redis会进行持久化

stop-writes-on-bgsave-error yes ##（持久化时），若出错，是否让服务器停止客户端请求，让用户意识到持久化出错了。
rdbcompression yes   ##持久化生成的rdb文件是否压缩 默认压缩  (压缩会消耗cpu资源)  但在主从复制时却可以节省主服务器的带宽。网络环境差的情况下可开启，网络环境好的情况下，cpu性能不好（或者从服务器数量多）就不要开启
rdbchecksum yes  ##持久化生成的rdb文件是否校验
dir ./   #rdb 文件保存的目录，默认当前文件夹
dbfilename dump.rdb   ##转储数据库的文件名

```

###### 5.主从复制配置 REPLICATION

```bash

```











###### 6.安全相关配置SECURITY

```bash
127.0.0.1:6379> config get requirepass  ##获得密码
1) "requirepass"
2) ""
127.0.0.1:6379> config set requirepass 123456  ##设置密码
OK
127.0.0.1:6379> config get requirepass   ##设置密码后 你的所有操作都报错，提示你要进行身份验证
(error) NOAUTH Authentication required.
127.0.0.1:6379> ping ##设置密码后 你的所有操作都报错，提示你要进行身份验证
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth 123456   ##进行身份验证后 可以进行操作
OK
```

###### 7.limit 限制

```bash
maxclients 10000  ##最大客户端连接数
 maxmemory <bytes> ##内存容量配置 单位byte
 
 
 maxmemory-policy noeviction  ##内存超过最大容量限制的策略方式   默认的过期策略是 volatile-lru
 maxmemory-policy 六种方式
        1、volatile-lru：只对设置了过期时间的key进行LRU（默认值） 
        2、allkeys-lru ： 删除lru算法的key   
        3、volatile-random：随机删除即将过期key   
        4、allkeys-random：随机删除   
        5、volatile-ttl ： 删除即将过期的   
        6、noeviction ： 永不过期，返回错误
```

###### 8.APPEND ONLY MODE AOF配置

```bash
appendonly no  ##默认不开启aof  默认使用rdb方式持久化
appendfilename "appendonly.aof"  ##aof方式持久化生成的默认文件名

### Redis supports three different modes: redis支持3中模式的同步方式
# appendfsync always    ## 只要发生一次写入就进行同步AOF  慢，但是更加安全  
appendfsync everysec  ##默认方式是每秒同步一次AOF   妥协的办法 可能会丢失1s的数据
# appendfsync no   ##不进行同步AOF，让系统在他想要清空内存的时候去同步AOF ，可以让redis新能跟好
```





### Redis持久化

​		redis是一个内存数据库，数据保存在内存中，但是我们都知道内存的数据变化是很快的，也容易发生丢失。幸好Redis还为我们提供了持久化的机制，分别是RDB(Redis DataBase)和AOF(Append Only File)。

**一、持久化流程**

既然redis的数据可以保存在磁盘上，那么这个流程是什么样的呢？

要有下面五个过程：

（1）客户端向服务端发送写操作(数据在客户端的内存中)。

（2）数据库服务端接收到写请求的数据(数据在服务端的内存中)。

（3）服务端调用write这个系统调用，将数据往磁盘上写(数据在系统内存的缓冲区中)。

（4）操作系统将缓冲区中的数据转移到磁盘控制器上(数据在磁盘缓存中)。

（5）磁盘控制器将数据写到磁盘的物理介质中(数据真正落到磁盘上)。

**redis为实现上面5个保存磁盘的步骤。提供了两种策略机制，也就是RDB和AOF**。

##### 二 RDB(Redis DataBase)  默认持久化策略

​	RDB其实就是把数据以**快照**的形式保存在磁盘上。什么是快照呢，你可以理解成把**当前时刻的数据拍成一张照片保存下来**。

​	RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘。也是默认的持久化方式，这种方式是就是将内存中数据以快照的方式写入到二进制文件中,默认的文件名为dump.rdb。

###### **1、save触发方式**

​	该命令会阻塞当前Redis服务器，执行save命令期间，Redis不能处理其他命令，直到RDB过程完成为止。具体流程如下：

![save方式持久化](C:\Users\91479\AppData\Roaming\Typora\typora-user-images\image-20201028063634740.png)

​		执行完成时候如果存在老的RDB文件，就把新的替代掉旧的。我们的客户端可能都是几万或者是几十万，这种方式显然不可取。

###### **2、bgsave触发方式**（全量保存）

​		执行该命令时，Redis会在后台异步进行(使用一个子进程)快照操作，快照同时还可以响应客户端请求。具体流程如下：

![bgsave方式](C:\Users\91479\AppData\Roaming\Typora\typora-user-images\image-20201028063853297.png)

​		具体操作是Redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束。阻塞只发生在fork阶段，一般时间很短。基本上 Redis 内部所有的RDB操作都是采用 bgsave 命令。同样如果存在旧的rdb文件也会覆盖掉。

###### **3、自动触发**

​		自动触发是由我们的配置文件来完成的。在redis.conf配置文件中，里面有如下配置，我们可以去设置：

**①save：**这里是用来配置触发 Redis的 RDB 持久化条件，也就是什么时候将内存中的数据保存到硬盘。比如“save m n”。表示m秒内数据集存在n次修改时，自动触发bgsave。

默认如下配置：

\#表示900 秒内如果至少有 1 个 key 的值变化，则保存save 900 1#表示300 秒内如果至少有 10 个 key 的值变化，则保存save 300 10#表示60 秒内如果至少有 10000 个 key 的值变化，则保存save 60 10000

不需要持久化，那么你可以注释掉所有的 save 行来停用保存功能。

**②stop-writes-on-bgsave-error ：**默认值为yes。当启用了RDB且最后一次后台保存数据失败，Redis是否停止接收数据。这会让用户意识到数据没有正确持久化到磁盘上，否则没有人会注意到灾难（disaster）发生了。如果Redis重启了，那么又可以重新开始接收数据了

**③rdbcompression ；**默认值是yes。对于存储到磁盘中的快照，可以设置是否进行压缩存储。

**④rdbchecksum ：**默认值是yes。在存储快照后，我们还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能。

**⑤dbfilename ：**设置快照的文件名，默认是 dump.rdb

**⑥dir：**设置快照文件的存放路径，这个配置项一定是个目录，而不能是文件名。

我们可以修改这些配置来实现我们想要的效果。因为第三种方式是配置的，所以我们对前两种进行一个对比：

###### 两种方式对比

![save和bgsave对比](C:\Users\91479\AppData\Roaming\Typora\typora-user-images\image-20201028064450898.png)

###### **4、RDB 的优势和劣势**

①、优势

（1）RDB文件紧凑，全量备份，非常适合用于进行备份和灾难恢复。

（2）生成RDB文件的时候，redis主进程会fork()一个子进程来处理所有保存工作，主进程不需要进行任何磁盘IO操作。

（3）RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。

②、劣势

RDB快照是一次全量备份，存储的是内存数据的二进制序列化形式，存储上非常紧凑。当进行快照持久化时，会开启一个子进程专门负责快照持久化，子进程会拥有父进程的内存数据，父进程修改内存子进程不会反应出来，所以在快照持久化期间修改的数据不会被保存，可能丢失数据



###### 注意：rdb触发机制还有 

1.执行 flushdb或者flushall命令

2.关闭redis服务器时。即shutdown 命令 ，当然可以使用shutdown no save 来跳过rdb





**三、AOF机制**

​		全量备份总是耗时的，有时候我们提供一种更加高效的方式AOF，工作机制很简单，redis会将每一个收到的写命令都通过write函数追加到文件中。通俗的理解就是日志记录。

###### **1、持久化原理**

![aof持久化原理](C:\Users\91479\AppData\Roaming\Typora\typora-user-images\image-20201028064752009.png)

每当客户端有一个写命令过来时，就直接保存在我们的AOF文件中。

###### **2、文件重写原理**

​			AOF的方式也同时带来了另一个问题。持久化文件会变的越来越大。为了压缩aof的持久化文件。redis提供了**bgrewriteaof**命令。将内存中的数据以命令的方式保存到临时文件中，同时会fork出一条新进程来将文件重写。

![bgwriteaof原理](C:\Users\91479\AppData\Roaming\Typora\typora-user-images\image-20201028065024766.png)

​		重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似。

###### **3、AOF也有三种触发机制**

（1）每修改同步always：同步持久化 每次发生数据变更会被立即记录到磁盘 性能较差但数据完整性比较好

（2）每秒同步everysec：异步操作，每秒记录 如果一秒内宕机，有数据丢失

（3）不同no：从不同步

![3中机制对比](C:\Users\91479\AppData\Roaming\Typora\typora-user-images\image-20201028065209179.png)

###### **4、优点**

（1）AOF可以更好的保护数据不丢失，一般AOF会每隔1秒，通过一个后台线程执行一次fsync操作，最多丢失1秒钟的数据。（2）AOF日志文件没有任何磁盘寻址的开销，写入性能非常高，文件不容易破损。

（3）AOF日志文件即使过大的时候，出现后台重写操作，也不会影响客户端的读写。

（4）AOF日志文件的命令通过非常可读的方式进行记录，这个特性非常适合做灾难性的误删除的紧急恢复。比如某人不小心用flushall命令清空了所有数据，只要这个时候后台rewrite还没有发生，那么就可以立即拷贝AOF文件，将最后一条flushall命令给删了，然后再将该AOF文件放回去，就可以通过恢复机制，自动恢复所有数据

###### **5、缺点**

（1）对于同一份数据来说，AOF日志文件通常比RDB数据快照文件更大

（2）AOF开启后，支持的写QPS会比RDB支持的写QPS低，因为AOF一般会配置成每秒fsync一次日志文件，当然，每秒一次fsync，性能也还是很高的

（3）以前AOF发生过bug，就是通过AOF记录的日志，进行数据恢复的时候，没有恢复一模一样的数据出来。

#### **四、RDB和AOF到底该如何选择**

​		选择的话，两者加一起才更好。因为两个持久化机制你明白了，剩下的就是看自己的需求了，需求不同选择的也不一定，但是通常都是结合使用。有一张图可供总结：

![两种持久化策略对比](C:\Users\91479\AppData\Roaming\Typora\typora-user-images\image-20201028065523260.png)



### redis做缓存时热点问题

#### 一、缓存穿透

###### 1.1什么是缓存穿透

​		比如，我们有一张数据库表，ID都是从1开始的(正数)：

但是可能有黑客想把我的数据库搞垮，每次请求的ID都是**负数**。这会导致我的缓存就没用了，请求全部都找数据库去了，但数据库也没有这个值啊，所以每次都返回空出去。

> 缓存穿透是指查询一个一定不存在的数据。由于缓存不命中，并且出于容错考虑，如果从数据库查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到数据库去查询，失去了缓存的意义。

 	 这就是缓存穿透，请求的数据在缓存中存在大量的不命中，而直接访问数据库的现象，且返回的数据也会逃过缓存的存储，导致一直不会命中缓存，因为缓存中本来就没有。

###### 1.2解决缓存穿透的方法

​		**1.设置过滤，对非法请求进行过滤 **由于请求的参数是不合法的(每次都请求不存在的参数)，于是我们可以使用**布隆过滤器(BloomFilter)或者压缩filter提前拦截**，不合法就**不让这个请求到数据库层**！

​		2.即使数据库返回值为空，也将他存到**缓存**，但设置一个较短的过期时间，下次请求还是会走缓存。

#### 二，缓存与数据库双写一致

###### 2.1 缓存和数据库数据不一致问题的原因

 	由于缓存是在查询数据库进行的缓存。那么在执行2次同样查询的中间时段，如果一个更新请求，改变了数据库的数据，那么此时第二次查询时，可能缓存中的数据并没有过期，这使得查询的结果和数据库结果不一致，这就是双学一致性问题。

```bash
1、读：
（1）先读cache，如果数据命中则返回
（2）如果数据未命中则读db
（3）将db中读取出来的数据入缓存
2、写：
（1）先淘汰cache
（2）再写db
```
![双写不一致](C:\Users\91479\AppData\Roaming\Typora\typora-user-images\image-20201028155904349.png)

**上图解析：** 写操作先执行1，删除缓存，再执行2，更新db；而读操作先执行3，读取cache数据，未找到数据时执行4，查询db。
**问题所在：** 写操作2没执行完时，读操作4执行了，则读到了脏数据到cache中，造成了cache和db的数据不一致问题。

##### 2.2 解决双写一致问题方案

方案1：Redis设置key的过期时间。
方案2：采用延时双删策略。
（1）先淘汰缓存
（2）再写数据库（这两步和原来一样）
（3）休眠1秒，再次淘汰缓存
这么做，可以将1秒内所造成的缓存脏数据，再次删除。(为何是1秒？需要评估自己的项目的读数据业务逻辑的耗时。这么做的目的，就是确保读请求结束，写请求可以删除读请求造成的缓存脏数据。当然这种策略还要考虑redis和数据库主从同步的耗时。)

### 三，缓存雪崩

##### 问题描述：

缓存在同一时间内大量键过期（失效），接着来的一大波请求瞬间都落在了数据库中导致连接异常。

##### 解决方案：

1、建立备份缓存，缓存A和缓存B，A设置超时时间，B不设值超时时间，先从A读缓存，A没有读B，并且更新A缓存和B缓存.

2， 并发量不是特别多的时候，使用最多的解决方案是加锁排队。

​		加锁排队只是为了减轻数据库的压力，并没有提高系统吞吐量。假设在高并发下，缓存重建期间key是锁着的，这是过来1000个请求999个都在阻塞的。同样会导致用户等待超时，这是个治标不治本的方法！

​		注意：加锁排队的解决方式分布式环境的并发问题，有可能还要解决分布式锁的问题；线程还会被阻塞，用户体验很差！因此，在真正的高并发场景下很少使用！



### 四，缓存击穿

##### 	现象

​	缓存击穿，是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个屏障上凿开了一个洞。

#####  解决方案

###### 1，"永远不过期"：

​	这里的“永远不过期”包含两层意思：

  (1) 从redis上看，确实没有设置过期时间，这就保证了，不会出现热点key过期问题，也就是“物理”不过期。

  (2) 从功能上看，如果不过期，那不就成静态的了吗？所以我们把过期时间存在key对应的value里，如果发现要过期了，通过一个后台的异步线程进行缓存的构建，也就是“逻辑”过期。

![image-20201028162156287](C:\Users\91479\AppData\Roaming\Typora\typora-user-images\image-20201028162156287.png)

​	从实战看，这种方法对于性能非常友好，唯一不足的就是构建缓存时候，其余线程(非构建缓存的线程)可能访问的是老数据，但是对于一般的互联网功能来说这个还是可以忍受。

###### 2.使用互斥锁(mutex key)：

​	这种解决方案思路比较简单，就是只让一个线程构建缓存，其他线程等待构建缓存的线程执行完，重新从缓存获取数据就可以了。

![](C:\Users\91479\AppData\Roaming\Typora\typora-user-images\image-20201028161829601.png)



###### 3."提前"使用互斥锁(mutex key)

​		在value内部设置1个超时值(timeout1), timeout1比实际的memcache timeout(timeout2)小。当从cache读取到timeout1发现它已经过期时候，马上延长timeout1并重新设置到cache。然后再从数据库加载数据并设置到cache中。

