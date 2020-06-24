# Redis

单线程，高性能的（key/value）内存数据库，基于内存运行并且支持持久化

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