# redis内容扩展

## 1.Pipeline

注意：使用Pipeline的操作是非原子操作

2.GEO
--------------------- 

GEOADD locations 116.419217 39.921133 beijin

GEOPOS locations beijin

GEODIST locations tianjin beijin km 	计算距离

GEORADIUSBYMEMBER locations beijin 150 km  通过距离计算城市

注意：没有删除命令  它的本质是zset  （type locations） 

所以可以使用zrem key member  删除元素

zrange key  0   -1  表示所有   返回指定集合中所有value

## 3.hyperLogLog

Redis 在 2.8.9 版本添加了 HyperLogLog 结构。

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

PFADD 2017_03_06:taibai 'yes' 'yes' 'yes' 'yes' 'no'

PFCOUNT 2017_03_06:taibai    统计有多少不同的值

1.PFADD 2017_09_08:taibai uuid9 uuid10 uu11

2.PFMERGE 2016_03_06:taibai 2017_09_08:taibai   合并

注意：本质还是字符串 ，有容错率，官方数据是0.81% 

## 4.bitmaps

setbit taibai 500000 0

getbit taibai 500000 

bitcount taibai

Bitmap本质是string，是一串连续的2进制数字（0或1），每一位所在的位置为偏移(offset)。
string（Bitmap）最大长度是512 MB，所以它们可以表示2 ^ 32=4294967296个不同的位。
