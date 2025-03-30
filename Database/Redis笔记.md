# Redis

## 简介

**传统的数据库交互**

- 用户 <-> 数据库
- 会出现大量的文件IO，磁盘IO
- 因为数据库内容是存放在磁盘中的，如果想访问数据库内容，那么一定需要大量的文件IO

**Redis数据库交互**

- 用户 <-> Redis <-> 数据库
- Redis作为用户和数据库的中间件，大大提高了数据库的性能。差一个数量级的快
- 中间件的作用通常都是作为一种缓存，来减少资源的消耗，提高效率。资源消耗如大量的文件io，大量的创建和销毁，大量的连接与释放。合理的使用中间件能够大大提高效率
- Redis作为一种**基于内存**的数据存储系统

**使用方式**

- CLI Command Line Interface
  - 命令行界面，通过命令行工具输入命令行使用Redis
- API
  - 应用程序接口，为语言提供便捷接口，使得程序只需要使用接口便可操纵Redis，而不是必须要用system
- GUI
  - 图形化界面，伟大无需多言
  - 常用的如 RedisInside，夸克有用的时候自己下载

**知识点**

- Redis中所有数据类型都是以键值对存储
- Redis中的key可以类比编程语言中的变量名，但内部实现不同
  - cpp中变量名会在最终编译时替换为虚拟地址，而并不是真正的变量名
  - Redis中的键名，确实真真正正的一个名字，保存在键值对中，使用时先查询找这个键值对然后才能使用
  - Redis的键值对在内部存储使用哈希表来实现，加快查询，每一个Redis中都会维护一个哈希表
  - 不愧是叫做Remote Dictionary Server

## `String`

- Redis中数据均是以键值对的形式存储的，需要指定一个键key一个值value，key表示的是一个数据类型，而非一个元素数据
- Redis中的键是区分大小写的，这个和MySQL是不同的，MySQL不区分大小写
- Redis中默认是以字符串来存储数据 
- 数据是二进制安全的，不会因为读出而导致数据错误
- 后面只是部分操作，所有操作记得查表

```shell
#设置数据内容
SET name string_content

#获取内容
GET name

#删除键值对
DEL name

#判断键值对是否存在
EXISITS name

#查看所有键，KEYS后面跟着一个pattern模式的字符
KEYS *
#查看以me结尾的键
KEYS *me

#删除所有键
FLUSHALL

#一个重要原因，Redis是基于内存的，我不能一直占有内存，所以需要设置一个占有时间
#查看键的过期时间
TTL name
#设置过期时间，设置十秒
EXPIRE name 10
#直接设置一个具有过期时间的键
SETEX name 5 content

#只有键不存在时，才设置值
SETNX name content
```



---

## `List`

- 用于存储和操作一组有顺序的数据
- 因为没有数组，线性表只能使用链表了，无奈
- 本质是一个双端队列，LR来判断IO位置
- 索引仍然是0开始

```shell
#添加到列表，key表示的是这个链表
LPUSH letter a

#从列表中读取数据
LRANGE letter starPos endPos
LRANGE letter 0 -1 #-1表示最后一个元素

#从列表中删除，删除前两个元素，真的好像栈
LPOP letter 2

#查看列表长度
LLEN letter

#删除指定范围外的元素
LTRIM letter startPos endPos
LTRIM letter 1 3 #0号和4号之后的元素被删除，（不包含4号，索引还是从0开始）
```

---

## `Set`

- 表示无序集合，元素之间没有联系和顺序，且元素不能重复
- 命令以S开头

```shell
#添加元素
SADD course math

#查看元素
SMEMBERS course

#判断是否存在元素
SISMEMBER course math

#删除元素
SREM course math

#集合运算
SINTER SUNION

```

---

## `SortedSet`

- 有序集合，也叫zset，和map很像，默认按照分数从小到大排序
- 每一个元素都会与一个浮点类型的分数相关联，然后按照这个分数排序，元素不可重复，但分数可以重复
- 命令以Z开头

```shell
#添加元素，注意这样是添加了四个元素，分数在前元素在后
ZADD result 690 清华 680 北大 670 复旦 670 上交

#查看有序集合元素
ZRANGE result 0 -1
ZRANGE result 0 -1 WITHSCORES #同时会输出分数

#查看某个分数
ZSCORCE result 清华

#查看元素排名，顺序
ZRANK result 清华
#查看排名，逆序
ZREVRANK result 清华

#删除元素
ZREM result 清华

```

---

## `Hash`

- 没啥可说的，一样，本质就是键值对定义对象，然后对象就是一个键值对组
- 只能是本质不是添加，而是在hash表中设置值（区别不大）
- 命令以H开头

```shell
#添加
HSET person 姚珅 22

#查看
HGET person 姚珅
HGETALL person

#删除
HDEL person 姚珅

#判断是否存在这个键值对
HEXISTS person 姚珅

#获取所有键
HKEYS person

#获取键值对数目
HLEN person
```

---

## `HyperLogLog`

- 用于做基数统计的算法
- 利用随机算法实现，牺牲精度换取更小的内存消耗，所以有一定的误差，用于不在乎这一点误差的时候，比如某个关键词搜索次数
- 命令以PF开头

---

## 发布订阅模式

- 但发布的内容无法持久化，无法进入日志

```shell
#订阅shen这个频道，其实就是读端
SUBSCRIBE shen

#发布shen这个频道，其实就是写端
PUBLISH shen redis #向shen中发布了'redis'这个内容

```

---

## 消息队列 Stream

- 一个轻量级的消息队列
- 命令以X开头

```
#添加，*表示自动生成id
XADD stream_name * course math
#手动添加id，第一个数是时间戳，第二个数是序号
XADD stream_name 1-0 course math

#查看
XLEN stream_name
XRANGE stream_name - +

#删除
XDEL stream_name id
XTRIM stream_name MAXLEN 0 #删除所有消息


```

## 地理位置 Geospatial

- 添加，查询，计算距离，查找范围内元素



---

## 位图 Bitmap











