# redis数据类型及api操作(http://redisdoc.com/)

## key

keys *  

scan  0 match  *  count  1

exists key 判断某个key是否存在

move key db  当前库就没有了，到指定的库中去了

expire key  为给定的key设置过期时间

ttl key 查看还有多少时间过期   -1表示永不过期  -2表示已过期

type key  查看key是什么类型

## 1.string

string是redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。

string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。

string类型是Redis最基本的数据类型，一个redis中字符串value最多可以是512M

set  key  value   设置key  value

get  key    查看当前key的值

del  key   删除key

append key  value   如果key存在，则在指定的key末尾添加，如果key存在则类似set

strlen  key  返回此key的长度

以下几个命令只有在key值为数字的时候才能正常操作

-----------------------------------------------------

incr  key  为执定key的值加一

decr  key  为指定key的值减一

incrby key  数值     为指定key的值增加数值

decrby key  数值     为指定key的值减数值

-----------------------------------------------------

getrange  key  0(开始位置)  -1（结束位置）    获取指定区间范围内的值，类似between......and的关系 （0 -1）表示全部

setrange key 1（开始位置，从哪里开始设置） 具体值    设置（替换）指定区间范围内的值

setex 键 秒值 真实值    设置带过期时间的key，动态设置。

setnx  key   value         只有在 key 不存在时设置 key 的值。

mset   key1   value  key2   value       同时设置一个或多个 key-value 对。

mget   key1   key 2    获取所有(一个或多个)给定 key 的值。

msetnx   key1   value  key2   value   同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。

getset   key    value  将给定 key 的值设为 value ，并返回 key 的旧值(old value)。



## 2.list

它是一个字符串链表，left、right都可以插入添加；
如果键不存在，创建新的链表；
如果键已存在，新增内容；
如果值全移除，对应的键也就消失了。
链表的操作无论是头和尾效率都极高，但假如是对中间元素进行操作，效率就很惨淡了。

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）。
它的底层实际是个链表

lpush  key  value1  value2  将一个或多个值加入到列表头部

rpush  key  value1  value2 将一个或多个值加入到列表底部

lrange key  start  end  获取列表指定范围的元素   （0 -1）表示全部

lpop key 移出并获取列表第一个元素

rpop key  移出并获取列表最后一个元素

lindex key index   通过索引获取列表中的元素 

llen 获取列表长度

 lrem key   0（数量） 值，表示删除全部给定的值。零个就是全部值   从left往right删除指定数量个值等于指定值的元素，返回的值为实际删除的数量

ltrim key  start(从哪里开始截)  end（结束位置） 截取指定索引区间的元素，格式是ltrim list的key 起始索引 结束索引



## 3.set

Redis的Set是string类型的无序，不能重复的集合。

sadd key value1 value 2 向集合中添加一个或多个成员

smembers  key  返回集合中所有成员

sismembers  key   member  判断member元素是否是集合key的成员

scard key  获取集合里面的元素个数

srem key value  删除集合中指定元素

srandmember key  数值     从set集合里面随机取出指定数值个元素   如果超过最大数量就全部取出，

spop key  随机移出并返回集合中某个元素

smove  key1  key2  value(key1中某个值)   作用是将key1中执定的值移除  加入到key2集合中

sdiff key1 key2  在第一个set里面而不在后面任何一个set里面的项(差集)

sinter key1 key2  在第一个set和第二个set中都有的 （交集）

sunion key1 key2  两个集合所有元素（并集）

## 4.hash

Redis hash 是一个键值对集合。
Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

kv模式不变，但v是一个键值对

类似Java里面的Map<String,Object>

hset  key  key value  向hash表中添加一个元素

hget key  key  向hash表中获取一个元素

hmset  key key1 value1 key2 value2 key3 value3 向集合中添加一个或多个元素

hmget key  key1 key2 key3  向集合中获取一个或多个元素

hgetall  key   获取在hash列表中指定key的所有字段和值

hdel  key  key1 key2 删除一个或多个hash字段

hlen key 获取hash表中字段数量

hexits key key  查看hash表中，指定key（字段）是否存在

hkeys  key 获取指定hash表中所有key（字段）

hvals key 获取指定hash表中所有value(值)

hincrdy key  key1  数量（整数）  执定hash表中某个字段加  数量  ，和incr一个意思

hincrdyfloat key key1  数量（浮点数，小数）  执定hash表中某个字段加  数量  ，和incr一个意思

hsetnx key key1 value1  与hset作用一样，区别是不存在赋值，存在了无效。

## 5.zset

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。
不同的是每个元素都会关联一个double类型的分数。
redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的,但分数(score)却可以重复。

zadd  key  score 值   score 值   向集合中添加一个或多个成员

zrange key  0   -1  表示所有   返回指定集合中所有value

zrange key  0   -1  withscores  返回指定集合中所有value和score

zrangebyscore key 开始score 结束score    返回指定score间的值

zrem key score某个对应值（value），可以是多个值   删除元素

zcard key  获取集合中元素个数

zcount key   开始score 结束score       获取分数区间内元素个数

zrank key vlaue   获取value在zset中的下标位置(根据score排序)

zscore key value  按照值获得对应的分数