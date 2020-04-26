# Redis数据类型

<!-- TOC -->

- [Redis数据类型](#redis数据类型)
    - [Redis Keys](#redis-keys)
    - [Redis支持的数据类型](#redis支持的数据类型)
        - [Strings](#strings)
            - [修改或查询key](#修改或查询key)
            - [设置key的过期时间](#设置key的过期时间)
        - [Lists](#lists)
            - [固定长度的list](#固定长度的list)
            - [list的阻塞操作](#list的阻塞操作)
            - [key的原子创建和删除](#key的原子创建和删除)
        - [Hashes](#hashes)
        - [Sets](#sets)
        - [Sorted Sets](#sorted-sets)
        - [Bit arrays/Bitmaps](#bit-arraysbitmaps)
        - [HyperLogLogs](#hyperloglogs)
        - [Streams](#streams)

<!-- /TOC -->

## Redis Keys

Redis keys是二进制安全(将输入保存为原始的、无任何特殊格式意义的数据流)，可以使用任何二进制序列作为key。空的字符串也是有效的key。

关于key有如下规则：

- key不要过长，不仅是占用内存，在查找时也会有一定的代价，更好的实践是通过对key进行hash；
- key太短也不建议，虽然占用更少的内存，但是可读性较差；
- key的命名建议选择结构命名，类似`object-type:id`的结构，比如`user:1000`，`.`或`-`比较常见，如`comment:1234:reply.to`或`comment:1234:reply-to`；
- key的大小最大值支持512MB

## Redis支持的数据类型

### Strings

`string`类型是Redis中最基础的类型。

`string`最基本的操作：

```
> set mykey somevalue
OK
> get mykey
"somevalue"
```

`SET`命令会覆盖已存在的key，并且还提供可选项参数：

```
# 只有key不存在的时候才执行
> set mykey newval nx
(nil)
# 只有key存在时才执行
> set mykey newval xx
OK
```

虽然值被存储为字符串，但它支持某些操作，例如原子加：

```
> set counter 100
OK
> incr counter
(integer) 101
> incr counter
(integer) 102
> incrby counter 50
(integer) 152
```

通过`MSET`, `MGET`单个命令设置或返回多个值来减少延迟：

```
> mset a 10 b 20 c 30
OK
> mget a b c
1) "10"
2) "20"
3) "30"
```

#### 修改或查询key

通过`EXISTS`命令查询指定key是否存在，`DEL`命令删除指定的key，返回值1或0取决于key是否存在：

```
> set mykey hello
OK
> exists mykey
(integer) 1
> del mykey
(integer) 1
> exists mykey
(integer) 0
```

`TYPE`命令用来返回key所存储的值类型，返回值：

- none: key不存在
- string: 字符串
- list: 列表
- set: 集合
- zet: 有续集
- hash: 哈希表

```
> set mykey x
OK
> type mykey
string
> del mykey
(integer) 1
> type mykey
none
```

#### 设置key的过期时间

对key设置一个过期时间，即ttl(time-to-live)，当达到key的过期时间，key会自动被销毁。

关于Redis的过期时间，你需要知道：

- 过期时间支持设置秒或者毫秒
- key过期时间的延迟在1毫秒之内
- key的过期时间保存在磁盘中，即使Redis Server服务停止过期时间依然在计算

```
> set key some-value
OK
> expire key 5
# 立即获取
> get key
"some-value"
# 超过5s时间
> get key
nil
```

`PERSIST`命令可以移除一个带有过期时间的key，即让key永不过期。也可以通过`SET`命令设置key的过期时间：

```
> set key 100 ex 10
OK
> ttl key
(integer) 9
```

要想设置key的过期时间是毫秒级，可以通过`PEXPIRE`命令：

```
> set key 100
OK
> pexpire key 1500
(integer) 1
# ttl命令返回秒单位
> ttl key
(integer) 2
# pttl命令返回精确毫秒数
> pttl key
(integer) 1499
```

### Lists

`List`这个词的本意是一系列元素的集合，在编程语言中，有`Array`(数组)或者`Linked List`(链表)的形式。

而Redis中的list数据结构是通过链表的方式，好处是哪怕list中的元素非常大，在头尾处添加元素的时间都是常量级，添加一个元素到长度为10的list和长度是100万的list所花费的时间是一致的。

当然链表有个最大的问题是，通过索引访问list中的元素不如数组来的快。如果你需要快速访问一个大集合中间的元素，那么`Sorted sets`会是一种解决方案。

`LPUSH`命令在list的头部插入一个新元素，`RPUSH`命令在list的尾部插入一个新元素，`LRANGE`命令返回list中指定区间的元素：

```
> rpush mylist A
(integer) 1
> rpush mylist B
(integer) 2
> lpush mylist first
(integer) 3
> lrange mylist 0 -1
1) "first"
2) "A"
3) "B"
```

`LPUSH`和`RPUSH`都支持可变参数，你可以传递任意长度的参数将元素添加到list中：

```
> rpush mylist 1 2 3 4 5 "foo bar"
(integer) 9
> lrange mylist 0 -1
1) "first"
2) "A"
3) "B"
4) "1"
5) "2"
6) "3"
7) "4"
8) "5"
9) "foo bar"
```

同样，可以通过`LPOP`或`RPOP`命令从list中移除一个元素：

```
> rpush mylist a b c
(integer) 3
> rpop mylist
"c"
> rpop mylist
"b"
> rpop mylist
"a"
> rpop mylist
(nil)
```

#### 固定长度的list

有些时候我们仅需要存储最新的一些元素，`LTRIM`命令可以仅保存指定长度的list元素，它有点类似`LRANGE`命令，同样给定一个范围，但是它会丢弃范围之外的元素。

```
> rpush mylist 1 2 3 4 5
(integer) 5
> ltrim mylist 0 2
OK
> lrange mylist 0 -1
1) "1"
2) "2"
3) "3"
```

#### list的阻塞操作

list数据结构可以很轻松地作为一个队列，并引用于producer/consumer(生产者消费者模式)：

- 生产者通过`LPUSH`添加元素到list中
- 消费者通过`RPOP`从list中取出元素

如果list是空的，那么`RPOP`命令返回`NULL`，那么消费者依然需要不停地请求，我们称之为轮询，它会带来两个问题：

1. Redis需要不停地处理无用的命令，因为list一直是空的，并且只会返回`NULL`；
2. 当然我们可以在得到`NULL`的结果时，增加一个延迟时间，但是在数量足够多的时候，又回到了第一个问题；

`BRPOP`和`BLPOP`命令在list为空时，它们可以被阻塞，只有在list添加了新的元素才返回给消费者，又或者超过了设置的等待时间。

```
# 从list中获取元素，没有元素一直等待，如果等待时间超过5s则直接返回
> brpop tasks 5
1) "tasks"
2) "do_something"
```

如果在指定时间没有任何元素，则返回一个`NULL`和等待时间；否则返回两个值，第一个值是返回元素所属的key，第二个值是返回元素值。由于这两个命令是支持可变参数，即可以同时等待多个list，所以返回值的第一个是list的名称。

#### key的原子创建和删除

当我们添加一个元素到不存在的list key中，Redis会帮我们创建一个空的list；或者从list中删除最后一个元素，Redis会自动销毁这个list。

上述说法除了list,同样适用于其他类型，如`Streams`, `Sets`, `Sorted Sets`和`Hashses`。

可以总结如下三条规则：

1. 当添加一个元素到key不存在的集合类型中，会先创建一个空的集合，再添加元素；
2. 当从集合中删除元素，如果集合为空，key会自动被销毁。`Streams`类型是特殊例外；
3. 对一个空的list，无论是执行只读命令如`LLEN`(返回list的长度)，还是删除list的元素，总是返回相同的结果；

例子1：

```
> del mylist
(integer) 1
> lpush mylist 1 2 3
(integer) 13
```

如果key已存在值，是不允许被执行不正确的命令：

```
> set foo bar
OK
> lpush foo 1 2 3
(error) WRONGTYPE Operation against a key holding the wrong kind of value
> type foo
string
```

例子2：

```
> lpush mylist 1 2 3
(integer) 3
> exists mylist
(integer) 1
> lpop mylist
"3"
> lpop mylist
"2"
> lpop mylist
"1"
> exists mylist
(integer) 0
```

例子3：

```
> del mylist
(integer) 0
> llen mylist
(integer) 0
> lpop mylist
(nil)
```

### Hashes

Redis hashes类型就是一种键值对类型：

```
> hmset user:1000 username antirez birthyear 1977 verified 1
OK
> hget user:1000 username
"antirez"
> hget user:1000 birthyear
"1977"
> hgetall user:1000
1) "username"
2) "antirez"
3) "birthyear"
4) "1977"
5) "verified"
6) "1"
```

hashes类型并没有明确的字段数量限制，取决于足够的内存。

`HMSET`设置多个键值对，`HGET`返回单个字段值，`HMGET`类似`HGET`返回键值对的数组。

有些命令可以操作hashes的单个字段，如`HINCRBY`命令：

```
> hincrby user:1000 birthyear 10
(integer) 1987
> hincrby user:1000 birthyear 10
(integer) 1997
```

操作hashes的命令有：

- HDEL key field [field ...]: 删除一个或多个字段
- HEXISTS key field: hash是否存在某个字段
- HGET key field: 获取某个字段值
- HGETALL key: 获取hash的所有字段及值
- HINCRBY key field increment: 对hash的某个字段整型加操作
- HINCRBYFLOAT key field incrment: 对hash的某个字段浮点加操作
- HKEYS key: 获取hash的所有字段
- HLEN key: 获取hash总的字段数量
- HMGET key field [field ...]: 获取指定字段值
- HMSET key field value [field value ...]: 对多个指定字段赋值
- HSET key field value [field value ...]: 对多个指定字段赋值（HSET一开始仅支持对单个字段赋值，功能和HMSET完全相同，在Redis 4.0.0后，考虑废弃）
- HSETNX key field value: 字段不存在才进行赋值
- HSETLEN key field: 获取hash指定字段的字符串长度
- HVALS key: 返回hash所有字段值
- HSCAN key cursor [MATCH pattern] [Count count]: 迭代获取hash中的键值对

```
> hset letters a 1 b 2 c 3 d 4
(integer) 4
> hscan letters 0
1) "0"
2) 1) "a"
   2) "1"
   3) "b"
   4) "2"
   5) "c"
   6) "3"
   7) "d"
   8) "4"
```

`HSCAN`命令同`SCAN`命令都是增量迭代元素集合，需要值得注意的地方：

1. count默认值是10；
2. 在不适用`MATCH`参数的情况下，在遍历`Sets`, `Hashes`, `Sorted Sets`或者key占用空间足够大使用`dicts`(哈希表)编码表示的时候，Redis会返回count或比count大的结果集；
3. 在遍历只包含`Integer`值的`Set`（`intsets`)，或者空间占用小使用`ziplists`(压缩列表)编码的`Hashes`, `Sorted Sets`集合，会返回集合所有元素，忽略count参数值；

```
> config get 'hash-max-ziplist-*'
1) "hash-max-ziplist-entries"
2) "512"
3) "hash-max-ziplist-value"
4) "64"
```

根据Redis的默认配置：

1. 当`Hashes`的键值对数量超过512时；
2. 当`Hashes`存在属性值长度超过64时；

上述两个条件，`Hashes`的编码方式都会从`ziplists`转为`dicts`。

### Sets

`Sets`是一组无序元素集合。

```
> sadd myset 1 2 3 b c a
(integer) 3
> smembers myset
1) "b"
2) "1"
3) "2"
4) "3"
5) "a"
6) "c"
```

检查`Sets`中是否存在某个元素：

```
> sismember myset 3
(integer) 1
> sismember myset 30
(integer) 0
```

以文章的标签为例，如果我们要给一篇ID为1000的文章打上tag ID 1, 2, 5, 27：

```
> sadd news:1000:tags 1 2 5 27
(integer) 4
```

当然也可以反向给tag打上所属的文章ID：

```
> sadd tag:1:news 1000
(integer) 1
> sadd tag:2:news 1000
(integer) 1
> sadd tag:5:news 1000
(integer) 1
> sadd tag:27:news 1000
(integer) 1
```

获取文章的所有tag IDs：

```
> smembers news:1000:tags
1. 5
2. 1
3. 27
4. 2
```

获取所有tag交集的文章：

```
> sinter tag:1:news tag:2:news tag:5:news tag:27:news
1) "1000"
```

文章ID1001和1000的并集，由于news:1001:tags是空集，所以结果集都是news:1000:tags：

```
> sunion news:1001:tags news:1000:tags
1) "1"
2) "2"
3) "5"
4) "27"
```

将文章ID1001和1000的并集，赋值给文章ID1002：

```
> sunionstore news:1002:tags news:1000:tags news:1001:tags
(integer) 4
> smembers news:1002:tags
1) "1"
2) "2"
3) "5"
4) "27"
# 查看结果集长度
> scard news:1002:tags
(integer) 4
# 随机移除一个元素
> spop news:1002:tags
"2"
```

### Sorted Sets

### Bit arrays/Bitmaps

### HyperLogLogs

### Streams

---

References:

- [data-types-intro](https://redis.io/topics/data-types-intro)
- [redis-command-scan](https://redis.io/commands/scan)