---
title: "Redis(1)——基础数据结构"
date: 2021-09-26T21:51:11+08:00
tags: [redis,数据库]
categories: [redis]
---

## String

```shell
> set name nic
> get name
> del name
> mset name1 nc name2 nc-77
> mget name1 name2
> setex name 5 nic // set+expire
> setnx name nic // set if name doesn't exist

> set age 20
> incr age // age = 21
> incrby age -1 // age = 20 
```

常用于缓存用户信息，将用户信息通过json序列化后转成字符串来缓存。

Redis中的string采用预分配，内部capacity一般高于实际存储长度len。当len>capacity时，内部会加倍扩容现有的空间。超过1MB时，每次扩容增加1MB空间。

## List

```shell
> rpush books golang python cpp
> rpop books
> lpush books java javascript
> lpop books
> llen books

> lindex books 1 // o(n)
> lrange books 0 -1 // 获取所有元素
> ltrim books start_index end_index // 保留该范围的元素
```

Redis中的list是双向链表而非数组，其底层并不是简单的linkedlist，而是一种叫快速链表（quicklist）的结构。

当list中元素较少时，会分配一块连续的内存，将所有的元素做压缩存储，称之为ziplist。ziplist内部原理具体见[todo]()

当元素过多时，redis会将linkedlist按段切分，每一段使用ziplist紧凑存储，多个ziplist之间使用双向指针连接。这样的结构称之为quicklist

![](https://img.nc-77.top/20210929141943.png)

## Hash

```shell
> hset books key value
> hmset books k1 v1 k2 v2 ...
> hget books key
> hmget books k1 k2
> hgetgall books
> hlen books
```

Redis中的hash相当于java中的hashMap，是一种无序字典，采用链地址法来解决hash冲突，具体结构如下图。

![](https://img.nc-77.top/20210929142453.png)

关于hash具体内部结构见[todo]()

## Set

```shell
> sadd books python golang
> spop books // 随机pop一个
> smembers books // 和插入的顺序不一致，set无序
> sismember books golang
> scard books
```

Redis中的set相当于java中的hashSet，内部键值对无序。常用于存储一些需要去重的数据。

## Zset

```shell
> zadd books 1 python 2 golang
> zrem books python
> zrange books 0 -1 // 按score顺序列出
> zreverange books 0 -1 // 按score逆序列出
> zcard books 
> zscore books golang // 获取指定value的score
> zrank books golang // 排名
> zrange books 0 -1 // 遍历zset
> zrangebyscore books 1 2 // 获取score为1-2区间的value
> zrangebyscore books 1 2 withsocre // 同时返回score
```

zset是内部有序的set，zset中每一个value都需要有一个score，代表该value的排序权重。

zset内部采用跳表的结构，来优化插入的效率。跳表具体结构分析见[todo]()

## 通用规则

list，hash，set，zset都属于容器型结构。共享两条规则：

1、create if not exists 

如果容器不存在，那就创建一个，再进行操作。

2、drop if no elements 

如果容器里元素没有了，那么立即删除容器，释放内存。

## 过期时间

过期以对象为单位，比如只能设置整个hash的过期时间，无法具体到某个key。

同时如果一个string设置了过期时间，使用set更改后，ttl将重置。

```
> set name nic 600
> ttl nic
```



## References

- 《Redis深度历险：核心原理与应用实践》
