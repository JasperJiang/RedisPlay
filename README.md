# Redis

## 是什么

单线程（不会有并发问题），高性能的（key/value）内存数据库，基于内存运行并且支持持久化。

## 能干嘛？

主要是用来做缓存，但不仅仅只能做缓存，比如：redis的计数器生成分布式唯一主键，redis实现分布式锁，队列，会话缓存，点赞，统计网站访问量。

## 安装

1. 解压
2. make
如果make报错的话 可能需要安装gcc 然后执行make distsclean 来删除之前build的缓存
3. make install

查看redis默认安装位置
/usr/local/bin

## 常用命令行工具

redis-benchmark 性能测试  
redis-check-aof 修复aof持久化文件  
redis-check-rdb 修复rdb持久化文件  
redis-cli 客户端工具  
redis-sentinel 启动哨兵节点  
redis-server  启动服务端  

redis 在启动的时候会在dir下找rdb文件去回复数据

## api操作[https://redisdoc.com/](https://redisdoc.com/)

## redis的持久化机制

说白了，就是在指定的时间间隔内，将内存当中的数据集快照写入磁盘，它恢复时是将快照文件直接读到内存

redis提供两种方式进行持久化：

- RDB

- AOF（append only file）

### RDB 

适合大数据到导入，性能好，但是会丢失数据

#### 是什么？

原理是redis会单独创建（fork）一个与当前进程一摸一样的子进程来进行持久化，这个子进程的所有数据（变量。环境变量，程序计数器等）都和原进程一摸一样，会先将数据写入到一个临时文件中，呆持久化结束了，再用这个临时文件替换上次持久化好都文件，整个过程中，主进程不进行任何的io操作，这就确保了极高的性能。（因为如果用主线程的话，其他IO操作会阻塞比如读写数据）

#### 什么时候触发rdb持久化？

- shutdown时，如果没有开启aof，会触发
- 执行命令save或者bgsave会触发。
    - save只管保存，其他不管，全部阻塞  
    - bgsave redis会在后台异步进行快照操作同时可以响应客户的请求
- 执行flushall命令  是清空所有的数据 （因为如果不做持久化那么如果意外宕机，flushall就失效了）
- 配置文件中默认的快照配置 （如果要关闭就删除这个配置）
```
save 900 1
save 300 10
save 60 10000
```
如果有主从复制的情况rdb关不了，因为他就是利用rdb来实现主从复制

### AOF 

解决数据丢失的问题，但是性能差，不适合大量数据导入

#### 是什么？

原理是将Redis的操作日志以追加的方式写入文件，读操作是不记录的，整体分为三步  

数据写入内存 --> 数据写入aof_buf --> 写入持久化文件。第二步到第三步什么时候执行根据配置文件触发机制  

**<font color=red>注意： aof持久化不会fork子进程</font>**

#### 为什么要AOF？

因为rdb是定时操作所以会导致数据丢失（比如在下一次持久化之前宕机了）

#### 如何开启

```
appendonly yes
```

#### 触发机制

```
appendfsync everysec
```

- no: 表示等操作系统进行数据缓存同步到磁盘（快、持久化没保证）等buffer区域满了才会触发没保证 绝对不能用
- always: 同步持久化，每次发生数据变更时，立即记录到磁盘（慢，安全）
- everysec: 表示没秒同步一次（默认值，很快，但是可能会丢失一秒内到数据）这个用得比较多

#### 启动流程

![redis_start](assets/redis_start.jpg)

**如果aof和rdb不一致。由于启动了aof就不会加载rbd来所以启动要先关闭aof，然后使用 config set appendonly yes来启动aof。然后再去该配置**

#### AOF文件格式

命令： SELECT 0
```
*2    //代表有几组命令
$6    //命令长度
SELECT
$1
0
```

#### AOF 重写

##### 什么是重写

去除AOF冗余数据，减少aof文件的大小

##### 触发机制
当AOF文件增长到一定大小到时候redis能够调用bgrewriteaof对日志文件进行重写。当AOF文件大小当增长率大于该配置项时自动开启重写（这里指超过愿大小当100%）（第一次触发之后） 

auto-aof-rewrite-percentage 100

当AOF文件增长到一定大小到时候redis能够调用bgrewriteaof对日志文件进行重写。当AOF文件大小大于该配置项时自动开启重写（第一次触发）

优化到话就调整这个参数，因为重写会消耗性能，所以不是很大到情况下就不需要重写

auto-aof-rewrite-min-size 64mb

**<font color="red">注意：重写操作是通过fork子进程来完成到，所有正常到aof不会fork子进程，触发来重写才会</font>**

![redis_start](assets/aof_rewrite.jpg)

##### 命令

bgrewriteaof

#### redis 4.0 之后混合持久化机制

4.0版本的混合持久化默认关闭的，通过aof-use-rdb-preamble配置参数控制，yes则表示开启，no表示禁用，5.0之后默认开启。

AOF重写前以rdb格式，重写之后用AOF格式

优点：混合持久化结合了RDB持久化 和 AOF 持久化的优点, 由于绝大部分都是RDB格式，加载速度快，同时结合AOF，增量的数据以AOF方式保存了，数据更少的丢失。

缺点：兼容性差，一旦开启了混合持久化，在4.0之前版本都不识别该aof文件，同时由于前部分是RDB格式，阅读性较差

## [redis数据类型](redis数据类型.md)

### string

string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象。

string类型是Redis最基本的数据类型，<font color="red">**一个redis中字符串value最多可以是512M**</font>

### list

它是一个字符串链表，left、right都可以插入添加； 如果键不存在，创建新的链表； 如果键已存在，新增内容； 如果值全移除，对应的键也就消失了。 链表的操作无论是头和尾效率都极高，但假如是对中间元素进行操作，效率就很惨淡了。

### set

Redis的Set是string类型的<font color="red">**无序**</font>，不能重复的集合。

### hash

Redis hash 是一个键值对集合。 Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

kv模式不变，但v是一个键值对

类似Java里面的Map<String,Object>

### zset 

有score来排序

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。 不同的是每个元素都会关联一个double类型的分数。 redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的,但分数(score)却可以重复。

## [redis内容扩展](redis数据类型扩展.md)

### GEO

经纬度距离运算

### hyperLogLog

用来做基数统计的算法。可以计算有多少个不同值，只要把IP地址加进去就可以计算网站的访问量

### bitmaps

位图，可以直接对内部二进制操作， 可以用来扩容 ```setbit key 10000 0```

bitcount 来统计数组中有多少个1

可以实现不保存数据库的情况下点赞功能

## [redis主从复制&哨兵](redis主从复制&哨兵.md)

从数据会被删除

![redis_master_slave](assets/redis_master_slave.png)

配从不配主

## [redis集群](redis集群.md)

## redis分布式锁

redisson

为了防止一台机器挂了可以使用redlock获取多个redis的锁 `"{node1}mylock"` 只会取花括号里面的来算hash槽位

## [缓存几大问题](redis缓存的几大问题.md)

### 缓存穿透

查询了一个不存在的数据（缓存和数据库中都没有都数据）

数据库没有，缓存也没有，这个适合不应该去查数据库

#### 方案

- 缓存空对象

    当没有在数据库查到数据库加一个空对象在redis缓存 

    - 缺点：
    
        - 如果黑客使用不同的key来攻击会导致redis有大量的空对象 

        - 超过过期事件后还是会去访问数据库

- 布隆过滤器： 实现原理是将值用n种hash算法来标记，最后来判断是否n个位置都为1

    布隆过滤器是一个集合里面有一亿个值，看key是否在集合中，占用内存特别小，底层使用位图实现

    有误判率（因为hash碰撞） 误判率（数组大小和hash函数的个数）越高消耗的内存越小

    ![bloomFilter](assets/bloomfilter.png)

    - 实现方法

        - Google 的 Guava 包里面有 BloomFilter 
            
            缺点：但是分布式不能用，创建的空间是占用java内存的

        - 可以用redis来实现

            1. 定义容错率和预计大小，

            2. 通过算法来创建多少大小的位和多少个hash函数

            3. 计算出hash值与数组大小取余
            
            4. 利用bitmap的方法去存redis

    - 缺点：

        - 维护麻烦，如果往数据库添加数据时需要添加到布隆过滤器中。 

        - 布隆过滤器 <font color="red">**不能删除**</font>。需要定时替换新的不布隆滤器

### 缓存雪崩

大部分数据失效/机器宕机

- 原因

    - 机器故障

    - 设置过期时间 --> 同时失效

- 解决方案

    - 高可用集群

    - 错开过期时间

### 缓存击穿

数据刚好失效/过期或者缓存中没有这条数据，此时来了并发访问 （数据库里有但是缓存里没有）

#### 解决方案

- 加（分布式）锁

    1. 查询缓存

    2. 加锁

        为了只放一个请求通过可以访问数据库

    3. 再一次查询缓存

        为了第二个请求可以在缓存中读到缓存，直接返回

    4. 释放锁

**note：如果使用java锁的话颗粒度比较大，但是用分布式锁可以定义key所以只有相同的key才会锁住**

## 删除策略

设置了过期时间的

redis使用了**定期删除**+**惰性删除**

- 定时删除

    有个定时器定时去查看哪个地址过期了就去删掉。

    以CPU内存换redis内存，很耗CPU

- 惰性删除

    在读的时候会去判断是否过期

    以redis内存换CPU内存，倒是redis内有大量已经过期的数据

- 定期删除

    1. redis启动的时候会读取配置文件hz的值，默认是10

    2. 每秒执行hz次serverCron() -> databasesCron() -> activeExpireCyle()

    3. activeExpireCyle() 对每一个expires[*]进行逐一检查，每次执行250ms/hz，不管执行多少次只给你250ms

        每一个redis 数据库（16个）都会有一个设有过期时间的区域这里称之为expires[*]

    4. 对某个expires[*]检测时，随机挑选N个key检查，默认N为20

        - 如果发现超时了，删除

        - 如果一轮中删除的key的数量超过了N的25%, 循环该过程

        - 如果一轮中删除的key小于等于N的25%， 检查下一个expires[*]

    current_db用于记录执行到了哪一个expires[*]，如果时间到了，下次会根据current_db继续执行

## 淘汰策略（逐出算法）

内存满了，没有过期的数据

新的vm机制如果内存不够用了会写入swap区（硬盘）

- 相关配置

    - maxmemory 
    
        redis最大内存限制，默认值0，表示不限制。生产环境一般根据需求设置，通常50%以上

    - maxmemory-policy 
        
        达到最大内存，对挑选出来对数据进行删除

        volatile -> 设置了过期时间的数据

        allkeys -> 所有数据

        - volatile-lru   利用LRU算法移除设置过过期时间的key (LRU:最近使用 Least Recently Used ) 
        
        - allkeys-lru   利用LRU算法移除任何key 
        
        - volatile-random 移除设置过过期时间的随机key 
        
        - allkeys-random  移除随机key 
        
        - volatile-ttl   移除即将过期的key(minor TTL) 
        
        - noeviction  不移除任何key，只是返回一个写错误 。默认选项

    - maxmemory-samples

        挑选几个值，然后值删除到新数据可以写入即可

## redis事务

一个命令执行的对队列，将一系列预定义命令包装成一个整体，就是一个队列。当执行的时候，**一次性**按照**添加顺序**依次执行，中间不会被**打断或者干扰**

1. 开启事务

    ```bash
    multi
    ```

2. 你的命令

3. 执行事务/取消事务

    ```bash
    exec/discard
    ```

如果事务中有**语法报错（加入队列时出错）eg：set 123**，则全都不会成功

如果事务中有**执行报错 eg: incr k1 (此时k1不是数字)**，这一条命令出错不会影响其他命令。不会回滚，需要程序员自己来回滚

## 监控key

一般会结合事务来使用

watch **在开启事务之前去监控**，如果事务中对key进行操作，如果别人修改了key则key不会别修改会返回nil

unwatch 取消监控

## redis发布订阅

- PSUBSCRIBE pattern [pattern ...]
    
    订阅一个或多个符合给定模式的频道。

- PUBSUB subcommand [argument [argument ...]]
    
    查看订阅与发布系统状态。

-	PUBLISH channel message

    将信息发送到指定的频道。

-	PUNSUBSCRIBE [pattern [pattern ...]]

    退订所有给定模式的频道。

-	SUBSCRIBE channel [channel ...]

    订阅给定的一个或多个频道的信息。

-	UNSUBSCRIBE [channel [channel ...]]

    指退订给定的频道。

## 数据库与缓存数据一致性

![数据库与缓存数据一致性](assets/数据库与缓存数据一致性.png)