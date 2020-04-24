# Redis数据类型

<!-- TOC -->

- [Redis数据类型](#redis数据类型)
    - [Redis Keys](#redis-keys)
    - [Redis支持的数据类型](#redis支持的数据类型)
        - [Strings](#strings)
        - [List](#list)
        - [Sets](#sets)
        - [Sorted Sets](#sorted-sets)
        - [Hashes](#hashes)
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

### List

### Sets

### Sorted Sets

### Hashes

### Bit arrays/Bitmaps

### HyperLogLogs

### Streams

---

References:

- [data-types-intro](https://redis.io/topics/data-types-intro)