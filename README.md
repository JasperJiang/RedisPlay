# Redis

单线程（不会有并发问题），高性能的（key/value）内存数据库，基于内存运行并且支持持久化

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

### RDB 

适合大数据到导入，性能好，但是会丢失数据

#### 是什么？

原理是redis会单独创建（fork）一个与当前进程一摸一样的子进程来进行持久化，这个子进程的所有数据（变量。环境变量，程序计数器等）都和原进程一摸一样，会先将数据写入到一个临时文件中，呆持久化结束了，再用这个临时文件替换上次持久化好都文件，整个过程中，主进程不进行任何的io操作，这就确保了极高的性能。（因为如果用主线程的话，其他IO操作会阻塞比如读写数据）

#### 什么时候触发rdb持久化？

- shutdown时，如果没有开启aof，会触发
- 执行命令save或者bgsave会触发。
    - save只管保存，其他不管，全部阻塞  
    - bgsave redis会在后台异步进行快照操作同时可以响应客户的请求
- 执行flushall命令  是清空所有的数据 （因为如果不做持久化那么如果意外当机，flushall就失效了）
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

因为rdb是定时操作所以会导致数据丢失（比如在下一次持久化之前当机了）

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
AOF重写前以rdb格式，重写之后用AOF格式
```
aof-use-rdb-preamble yes
```
