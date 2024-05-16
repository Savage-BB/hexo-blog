---
title: Redis入门
date: 2022-07-26
description: 介绍Redis基础的使用
categories: 数据库
tags: 
  - Redis
---
# Redis入门

# Redis安装和启动

• GitHub Windows版本维护地址：[https://github.com/tporadowski/redis/releases](https://github.com/tporadowski/redis/releases)

通过链接下载zip文件，解压后在该文件夹，打开cmd窗口输入 

> redis-server.exe redis.windows.conf
> 

即可使用`redis.windows.conf`配置文件，打开redis服务器 

![Untitled](/pic/Redis%E5%85%A5%E9%97%A8%20c43303fb41ab4a8bb7e0dece858cb898/Untitled.png)

双击redis-cli.exe 即可操作redis数据库

![Untitled](/pic/Redis%E5%85%A5%E9%97%A8%20c43303fb41ab4a8bb7e0dece858cb898/Untitled%201.png)

![Untitled](/pic/Redis%E5%85%A5%E9%97%A8%20c43303fb41ab4a8bb7e0dece858cb898/Untitled%202.png)

# Redis 基本操作

在Redis下，数据库是由一个整数索引标识，而不是由一个数据库名称。 默认情况下，我们连接Redis数据库之后，会使用0号数据库，我们可以通过Redis配置文件中的参数来修改数据库总数，**默认为16个**。

我们可以通过`select`语句进行切换：

> select 序号;
> 

```
127.0.0.1:6379> select 15
OK
127.0.0.1:6379[15]> select 16
(error) ERR invalid DB index
```

## **数据操作**

我们来看看，如何向Redis数据库中添加数据：

```bash
set <key> <value>
-- 一次性多个
mset [<key> <value>]...
```

我们可以通过键值获取存入的值：

```
get <key>

127.0.0.1:6379> get name
"OTTO"
```

所有存入的数据默认会以**字符串**的形式保存，键值具有一定的命名规范，以方便我们可以快速定位我们的数据属于哪一个部分，比如用户的数据：

```
-- 使用冒号来进行板块分割
set admin:name PILL

127.0.0.1:6379> get admin:name
"PILL"
```

设置数据的过期时间：

```bash
set <key> <value> EX 秒
set <key> <value> PX 毫秒
```

当数据到达指定时间时，会被自动删除。我们也可以单独为其他的键值对设置过期时间：

```bash
expire <key> 秒
```

查询某个键值对的过期时间还剩多少：

```bash
ttl <key>
-- 毫秒显示
pttl <key>
-- 转换为永久
persist <key>
```

```
127.0.0.1:6379> set name LUCY ex 15
OK
127.0.0.1:6379> get name
"LUCY"
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> get name
(nil)   //获取结果为  **nil**  表示未获取到数据。
```

**ttl会返回一个integer类型的数据来表示剩余时间，通过persist可以转换为永久数据**

![Untitled](/pic/Redis%E5%85%A5%E9%97%A8%20c43303fb41ab4a8bb7e0dece858cb898/Untitled%203.png)

删除键

```
del <key>...
```

删除命令可以同时拼接多个键值一起删除。删除后返回integer表示删除数据的个数。

![Untitled](/pic/Redis%E5%85%A5%E9%97%A8%20c43303fb41ab4a8bb7e0dece858cb898/Untitled%204.png)

当我们想要查看数据库中所有的键值时：

```
keys *
```

![Untitled](/pic/Redis%E5%85%A5%E9%97%A8%20c43303fb41ab4a8bb7e0dece858cb898/Untitled%205.png)

也可以查询某个键是否存在：

```
exists <key>...
```

还可以随机拿一个键：

```
randomkey
```

我们可以将一个数据库中的内容移动到另一个数据库中：（类似剪切功能）

```
move <key> 数据库序号
```

修改一个键为另一个键：

```
rename <key> <新的名称>
```

如果存放的数据是一个数字，我们还可以对其进行自增自减操作：

```
-- 等价于a = a + 1
incr <key>
-- 等价于a = a + b
incrby <key> b
-- 等价于a = a - 1
decr <key>
```

操作结束会返回计算结果

![Untitled](/pic/Redis%E5%85%A5%E9%97%A8%20c43303fb41ab4a8bb7e0dece858cb898/Untitled%206.png)

最后是查看值的数据类型：

```
type <key>
```

# 数据类型

## Hash

它比较适合存储类这样的数据，由于值本身又是一个Map，因此我们可以在此Map中放入类的各种属性和值，以实现一个Hash数据类型存储一个类的数据。

我们可以像这样来添加一个Hash类型的数据：

```
hset <key> [<字段> <值>]...
```

我们可以直接获取：

```
hget <key> <字段>
-- 如果想要一次性获取所有的字段和值
hgetall <key>
```

![Untitled](/pic/Redis%E5%85%A5%E9%97%A8%20c43303fb41ab4a8bb7e0dece858cb898/Untitled%207.png)

同样的，我们也可以判断某个字段是否存在：

```
hexists <key> <字段>
```

![Untitled](/pic/Redis%E5%85%A5%E9%97%A8%20c43303fb41ab4a8bb7e0dece858cb898/Untitled%208.png)

删除Hash中的某个字段：

```
hdel <key>
```

我们现在想要知道Hash中一共存了多少个键值对（类的属性的个数）：

```
hlen <key>
```

![Untitled](/pic/Redis%E5%85%A5%E9%97%A8%20c43303fb41ab4a8bb7e0dece858cb898/Untitled%209.png)

我们也可以一次性获取所有字段的值： 

```
hvals <key>
hkeys //取所有键
```

唯一需要注意的是，Hash中只能存放字符串值，不允许出现嵌套的的情况。

## **List**

我们可以直接向一个已存在或是不存在的List中添加数据，如果不存在，会自动创建：

```
-- 向列表头部添加元素
lpush <key> <element>...
-- 向列表尾部添加元素
rpush <key> <element>...
-- 在指定元素前面/后面插入元素
linsert <key> before/after <指定元素> <element>
```

获取元素也非常简单：

```
-- 根据下标获取元素
lindex <key> <下标>
-- 获取并移除头部元素
lpop <key>
-- 获取并移除尾部元素
rpop <key>
-- 获取指定范围内的
lrange <key> start stop
```

注意下标可以使用负数来表示从后到前数的数字（Python：搁这儿抄呢是吧）:

```
-- 获取列表a中的全部元素
lrange a 0 -1
```

没想到吧，push和pop还能连着用呢：

```
-- 从前一个数组的最后取一个数出来放到另一个数组的头部，并返回元素
rpoplpush 当前数组 目标数组
```

它还支持阻塞操作，类似于生产者和消费者：

```
-- 如果列表中没有元素，那么就等待，如果指定时间（秒）内被添加了数据，那么就执行pop操作，如果超时就作废，支持同时等待多个列表，只要其中一个列表有元素了，那么就能执行
blpop <key>... timeout
```

**可以为阻塞弹出设置时间，当list为空时 ，程序会在该时间内保持请求状态，只要有元素插入，则会立刻弹出。超过请求时间都没有请求到元素则会退出。**

## **Set、SortedSet**

Set集合其实就像Java中的HashSet一样它不允许出现重复元素，不支持随机访问，但是能够利用Hash表提供极高的查找效率。

向Set中添加一个或多个值：

```
sadd <key> <value>...
```

查看Set集合中有多少个值：(返回个数)

```
scard <key>
```

判断集合中是否包含：

```
-- 是否包含指定值
sismember <key> <value>   //(返回值为1/0)
-- 列出所有值
smembers <key>
```

集合之间的运算：

```
-- 集合之间的差集
sdiff <key1> <key2>
-- 集合之间的交集
sinter <key1> <key2>
-- 求并集
sunion <key1> <key2>
-- 将集合之间的差集存到目标集合中
sdiffstore 目标 <key1> <key2>
-- 同上
sinterstore 目标 <key1> <key2>
-- 同上
sunionstore 目标 <key1> <key2>

```

移动指定值到另一个集合中：

```
smove <key> 目标 value

```

移除操作：

```
-- 随机移除一个幸运儿
spop <key>
-- 移除指定
srem <key> <value>...

```

那么如果我们要求Set集合中的数据按照我们指定的顺序进行排列怎么办呢？这时就可以使用SortedSet，它支持我们为每个值设定一个分数，分数的大小决定了值的位置，所以它是有序的。

我们可以添加一个带分数的值：

```
zadd <key> [<value> <score>]...

```

同样的：

```
-- 查询有多少个值
zcard <key>
-- 移除
zrem <key> <value>...
-- 获取区间内的所有
zrange <key> start stop

```

由于所有的值都有一个分数，我们也可以根据分数段来获取：

```
-- 通过分数段查看
zrangebyscore <key> start stop [withscores] [limit]
-- 统计分数段内的数量
zcount <key>  start stop
-- 根据分数获取指定值的排名
zrank <key> <value>

```

[https://www.jianshu.com/p/32b9fe8c20e1](https://www.jianshu.com/p/32b9fe8c20e1)

有关Bitmap、HyperLogLog和Geospatial等数据类型，这里暂时不做介绍，感兴趣可以自行了解。

> B站 清空の霞光