---
title: "Redis(2)——简单应用"
date: 2021-09-29T19:40:41+08:00
tags: [redis,数据库]
categories: [redis]
---

## 前言

这是《Redis深度历险》这本书的第二篇读书笔记，不得不说，这本书行文幽默，很容易就读进去，而且还是彩印的。强烈推荐一波~

Redis除了广泛作为数据缓存的中间件以外，还可用于构建分布式锁，延迟队列，布隆过滤器，限流等，下面将通过一些简单的应用场景来展示redis在这些方面的应用，体会redis的强大之处。

## 分布式锁

### 简单应用

看下面这个场景，模拟了当多个并发程序从账户余额中扣款的情况，正常逻辑期望最终余额为0。但运行发现最终结果大概率不为0，这种典型的并发问题称之为丢失更新。

```go
var Account = 1000
func getAccount()int{
	return Account
}
func saveAccount(account int){
	Account = account
}
func main() {
	for i:=0;i<1000;i++{
		wg.Add(1)
		go func() {
			defer wg.Done()
			account :=getAccount()
			account -=1
			saveAccount(account)
		}()
	}
	wg.Wait()
	fmt.Println(Account)
}
```

当然，在这段程序中我们可以添加sync.mutex来保障线程安全，不过如果是分布式的情况呢，怎么来共享这个锁，答案就是Redis。

分布式锁的本质就是通过Redis来对进程之间的共享资源占个坑，当别的进程试图访问时，就只好放弃或者等待。

Redis中有setnx（set if not exist）这一原子操作来实现占坑，最后通过del指令释放。

```go
var (
	Account = 1000
	wg      sync.WaitGroup
	rdb     *redis.Client
	ctx = context.Background()
)

func getAccount() int {
	return Account
}
func saveAccount(account int) {
	Account = account
}

func Lock(key string) {
	for { // 自旋，等待资源解锁
		ok, err := rdb.SetNX(ctx, key, 1, time.Second).Result()
		if err != nil {
			log.Println(err)
		}
		if ok {
			break 
		}
	}
}
func UnLock(key string) {
	_, err := rdb.Del(ctx, key).Result()
	if err != nil {
		log.Println(err)
	}
}

func main() {
	rdb = redis.NewClient(&redis.Options{
		Addr: "localhost:6379",
		DB:   0,
	})
	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			Lock("lock:account")

			account := getAccount()
			account -= 1
			saveAccount(account)

			UnLock("lock:account")
		}()
	}

	wg.Wait()
	fmt.Println(Account)
}
```

### 死锁

在使用Redis分布式锁时，还需考虑以下几种情况。

如果在某个进程拿到锁后执行任务时出错了，可能会导致锁资源没被释放，这就会导致死锁，因此，在上锁时需要添加一个过期时间。

同时，在Redis v2.8后，添加了setnx的额外参数，使得setnx和expire可以合并为一个原子指令，防止程序在setnx和expire之间时挂掉而导致的死锁。

对于执行时间较长的任务，也不建议使用Redis分布锁。因为如果程序在加锁和释放锁之间的逻辑执行得太长，超出了锁的超时限制，会导致后来的进程提前获取了这把锁，产生一些不可预计的错误。



## Hyperloglog

### 简单应用

看下面一个经典场景：统计一个网站上每天的UV（同一个用户一天之内的多次访问至多算一次）。

很容易想到一种方案，将每个访问ip添加到redis的set中，最后只需要scard获取UV。当网站访问人数较少时，这不失为一个简单易行的方案，但如果统计的是一个爆款网站，每天访问有上百万，这样去使用set空间消耗将是巨大的。同时对老板而言，100万和101万的数字没有什么区别。

Redis中的Hyperloglog就是这种场景下的一个解决方案。Hyper提供两种指令pfadd和pfcount

```bash
> pfadd website user1
> pfadd website user2
> pfcount website
```

当添加到10w个数据时，Hyperloglog的误差率也只有0.277%，同时仅需要占据12KB的存储空间，相较于存储10w个用户的set简直是九牛一毛。



## 布隆过滤器

### 简单应用

刚刚提到的Hyperloglog可以计算一个集合内的元素数量，但对某个元素是否在集合内却无能为力。

看下面一个经典场景：我们每天在b站，抖音上愉快地刷短视频时，很少会刷到重复的视频。这种去重是如何做到的？如果历史记录存在关系型数据库中，那么频繁地去重显然会把数据库压垮。用缓存？将所有历史记录缓存起来，当时间一长，一个月还可以，一年得浪费多大的存储空间。因此，就需要Bloom Filter（布隆过滤器，以下简称为bloom）出场了。

何为bloom？可以简单理解为一个不精确的set。bloom有两个基本指令bf.add和bf.exists。为什么说不精确，因为bloom说某个值存在时，它可能不存在，但当bloom说它不存在时，它就一定不存在。

在默认情况下，bloom的误判率约为1%左右，默认的initial_size为100。可以通过bf.reserve来设置initial_size，当需要的空间很大时，应提前设置inital_size，防止误判率上升。

错判率为1%时，一个元素需要的空间大约为10bit，相较于set存储空间优势明显。

### 原理分析

说了这么多，如此巧妙的数据结构到底是如何实现的呢？

![](https://img.nc-77.top/20210929215936.png)

当向bloom中添加key时，bloom会使用多个hash函数（f,g,h）对key取得一个value，再将value对数组长度取模得到位置，将该位置上的0置1。

因此，当查询某个key是否存在时，bloom会检查对应位置是否全为1，如果有一个为0，则返回不存在。可见，由于多个key hash后的位置会有重叠，因此当bloom回答不存在时，则一定不存在，而回答存在key3时，其实可能是key1+key2混合的误判，其实是不存在的。



## 限流

### 简单限流

对一个接口请求限流是一种常见的需求。比如需要对一个用户在一个时间段内的特定动作进行次数限制。

将该需求具象成一个limiter+http接口的形式，可以先思考如何合理使用redis中的数据结构来实现Allow（）。

```go
	// 1s内同一用户最多请求5次该接口
	limiter:=&Limiter{
		period:   time.Second,
		capacity: 5,
	}
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        userId:=0
		if limiter.Allow(0){
			fmt.Printf("请求成功，当前时间：%s\n", time.Now().Format("2006-01-02 15:04:05"))
		}
	})
	_ = http.ListenAndServe(":8081",nil)
```

还记得redis中的zset吗，其特性是可以根据score排序。利用该特性，我们可以这样来实现

1. 将userId作为zset的key，请求时间戳作为score

2. 每次有请求到来时，将其放入对应key的zset中，记录时间戳now为score

3. 保留score在（now-period，now）之间的member，相当于移除了已经失效的行为记录

4. 统计当前zset中member数目，如果小于限流数，则处理该请求，否则拒绝

5. 最后设置zset过期时间，防止冷用户长时间占用内存

   ```go
   type Limiter struct {
   	period time.Duration
   	capacity int64
   }
   
   func (l *Limiter)Allow(userId int)bool{
   	// 使用userId作为key，time作为score
   	key:=strconv.Itoa(userId)
   	now:=time.Now()
   	// 记录行为
   	rdb.ZAdd(ctx,key,&redis.Z{Score:float64(now.Unix()) ,Member: now})
   	// 移除无效的行为记录，框定滑动窗口
   	rdb.ZRemRangeByScore(ctx,key,"0", strconv.FormatInt(now.Add(-l.period).Unix(), 10))
   	// 统计当前数目
   	curCount:=rdb.ZCard(ctx,key).Val()
   	// 设置过期时间，防止冷用户占用空间
   	rdb.Expire(ctx,key,l.period+time.Second)
   
   	return curCount<=l.capacity
   }
   ```

   来压测一下~

   ```go
   func getApi() {
   	api := "http://localhost:8081/"
   	res, err := http.Get(api)
   	if err != nil {
   		panic(err)
   	}
   	defer res.Body.Close()
   
   	if res.StatusCode == http.StatusOK {
   		fmt.Printf("get api success\n")
   	}
   }
   func Benchmark_Main(b *testing.B) {
   	for i:=0;i<b.N;i++ {
   		getApi()
   	}
   }
   ```

   结果如下，可以看到如预期每秒钟只响应了同一用户的5次请求，其余都被过滤掉了。

   <img src="https://img.nc-77.top/20211025225752.png" style="zoom:50%;"   />



## References

- 《Redis深度历险：核心原理与应用实践》

