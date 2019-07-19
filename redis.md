### redis

#### 1.概念

redis是一款高性能的NOSQL系列的非关系型数据库

##### 关系型数据库mysql~oracle...

1.数据之间有关联关系

2.数据储存在硬盘的文件上

##### 非关系型数据库(NOSQL)redis~hbase...

1.数据之间没有关联关系

2.数据储存在内存中

#### 2.redis数据结构

redis储存的是：key,value格式的数据，其中kry都是字符串，value有五种不同的数据结构

##### 	字符串类型 string

​		1.存储：set key value

​		2.获取：get key

​		3.删除：del key

##### 	哈希类型 hash : map格式

​		1.存储：set key value

​		2.获取：

​			*hget key field：获取指定的filed对应的值

​			*hegetall key：获取所有的filed和value

​		3.删除：hdel key filed

##### 	列表类型 list		

​	可以添加一个元素到列表的头部或者尾部

​		1.添加：

​			1.lpush key value：将元素加入列表左边

​			2.rpush key value：将元素加入列表右边

​		2.获取：

​			lrange key start end ：范围获取

​		3.删除：

​			lpop key ：删除列表最左边的元素，并将元素返回

​			rpop key ；删除列表最右边的元素，并将元素返回

##### 	集合类型 set

​			不允许有重复元素

​				1.存储：sadd key value

​				2.获取：smembers key：获取set集合中所有元素

​				3.删除：srem key value：删除set集合中所有元荤

##### 	有序集合类型 sortedset

​		不允许有重复元素，且元素有序

​			1.存储：zadd key score value：

​			2.获取：zrange key start end

​			2.删除：zrem key value

##### 	通用命令

​		keys * ：查询所有的健

​		type key：获取键对应的value的类型

​		del key： 删除指定的key value

#### 3.持久化

​	redis是一个内存数据库，当redis服务器重启，获取电脑重启，数据会丢失，我们可以将redis内存中的数据持久化保存到硬盘的文件中。

#### 	redis持久化机制：

##### 		RDB:默认方式，不需要配置，默认使用这种配置

​			在一定时间间隔内，检测key变化情况，然后持久化数据

###### 				编辑redis.windows.conf文件

​					save 900 1 

​					save 300 10

​					save 60 10000

###### 				2.重启redis服务器，并指定配置文件名称

​					redis-server.exe redis.windows.conf

##### 		2.AOF：日志记录方式，可以记录每一条操作，可以每一次命令操作后，持久化数据

​	1.编辑redis.windows.conf文件

​		appendonly no(关闭aof)->appendonly yes（开启aof）

​		#appendfsync always ： 每一次操作都进行持久化

​		appendfsync everysec ： 每隔一秒进行一次持久化

​		#appendfsync no ： 不进行持久化

#### java客户端 Jedis

是一款java操作redis数据库的工具

##### 	1.下载jedis jar包

##### 	2.使用

​		//获取连接

​		Jedis jedis = new Jedis(“localhost”，6379)；

​		//操作

​		jedis.set("username","zhangsan");

​		//关闭连接

​		jedis。close();

##### 	Jedis操作各种jedis中的数据结构

​		