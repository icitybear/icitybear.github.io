---
title: "redis缓存常见问题" #标题
date: 2024-08-20T10:47:18+08:00 #创建时间
lastmod: 2024-08-20T10:47:18+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- redis
- 缓存
keywords: 
- 
description: "" #描述 每个文章内容前面的展示描述
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true #是否展示评论 有自带的扩展成twikoo
showToc: true # 显示目录 文章侧边栏toc目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: "" #图片路径：posts/tech/文章1/picture.png
    caption: "" #图片底部描述
    alt: ""
    relative: false

# reward: true # 打赏
mermaid: true #自己加的是否开启mermaid
---
# 缓存的过期策略和key的失效策略
- <font color="red">[b站视频](https://b23.tv/fvdQV2X)</font>
- FIFO（先进先出） LRU（时间） LFU（频率） (hash和双向链表实现)
- 定时任务，过期时间（同步或异步），访问时删除    redis的key失效

# 缓存穿透（针对单个key）
缓存穿透是查询一个根本不存在的数据，缓存层和存储层都不会命中，但是出于容错的考虑，如果从存储层查不到数据则不写入缓存层。
每次从缓存中都查不到数据，而需要查询数据库，同时数据库中也没有查到该数据，也没法放入缓存。也就是说，每次这个用户请求过来的时候，都要查询一次数据库。

- <font color="red">倘若读操作频繁请求 db 中不存在的数据，那么该数据自然也无法写入 cache，最终所有请求都会直击 db，导致 db 压力较大.</font>
![alt text](image1.png)
解决方案：
## 校验参数和布隆过滤器bitmap
- 存储层之上额外封装一层布隆过滤器，用于提前过滤大部分不存在的 key
- 缺点：
  1. 存在误判的情况。
  2. 存在数据更新问题。


## 设置特殊值 
关键点是不管从数据库有没有查到数据，都将结果放入缓存中，只是如果没有查到数据，缓存中的值是空(或者默认值)
<font color="red">这样第二次到缓冲中获取就有值了，而不会继续访问数据库。</font>
- 缺点：
  1. <font color="red">无效键值太多，占用缓存空间。</font>
  2. 缓存层和存储层的数据会有一段时间的不一致。（数据一致性的问题）
{{< innerlink src="posts/tech/redis1.md" >}}

# 缓存击穿（针对单个热门key）

某个时刻，该商品到了过期时间失效了。
此时，如果有大量的用户请求同一个商品，但该商品在缓存中失效了，一下子这些用户请求都直接怼到数据库，可能会造成瞬间数据库压力过大，而直接挂掉。
- go防止缓存击穿之共享内存调用SingleFlight
{{< innerlink src="posts/tech/go39.md" >}}

## 加锁 互斥锁
只能允许一个请求访问处理重建缓存，其他请求都是等待重建缓存的请求执行完，重新从缓存获取数据即可。  控制查询数据库的线程访问
构建缓存过程出现问题或者时间较长，可能会存在死锁的风险。 缺点降低了吞吐量

## 自动续期 （公共缓存的数据 筛选个别请求进行续期）  预热脚本
在key快要过期之前，就自动给它续期  定时任务？ 缺点：脚本定时续期 
在秒杀活动开始前，我们先用一个程序提前从数据库中查询出商品的数据，然后同步到缓存中，提前做预热。

## 缓存不失效（永久）
在缓存中我们可以不设置过期时间。比较少用

## 高可用（避免单点失效）
比如：如果使用了redis，可以使用哨兵模式，或者集群模式 （避免单点失效）

# 缓存雪崩（大量key）
- <font color="red">缓存雪崩是缓存击穿的升级版，缓存击穿说的是某一个热门key失效了，而缓存雪崩说的是有多个热门key同时失效</font>
![alt text](image2.png)
1. 有大量的热门缓存，同时失效。会导致大量的请求，访问数据库。而数据库很有可能因为扛不住压力，而直接挂掉。
2. 缓存服务器down机了，可能是机器硬件问题，或者机房网络问题。总之，造成了整个缓存的不可用。

## 过期时间加随机数 
缓存雪崩问题的常用解决思路是切断问题的直接导火索，对 cache key 的过期时间进行打散，比如可以在预设过期时间的基础上，加上随机扰动值，因此来<font color="red">避免大面积 cache key 同时失效的情形.</font>
## 高可用 
比如：如果使用了redis，可以使用哨兵模式，或者集群模式

## 服务降级 
缓存降级 到db的时候加个限流  (对应go的限流包)

# 缓存倾斜
分布式一致性时会出现的问题
{{< innerlink src="posts/tech/redis3.md" >}}

# 多级缓存架构和热点key检测的设计与分析
- <font color="red">[b站视频](https://b23.tv/pSJjN34)</font>
- 本地缓存（服务端-单体内存缓存和客户端缓存）
- 存储介质的缓存

# redis一些最佳实践
## 大key新增数据，hash表的rehash机制导致
- [【有道云笔记】redis内存使用率异常增长问题](https://note.youdao.com/s/NPBPDw6)
## 查询大key,hash拆分为string
[【有道云笔记】redis大key拆分](https://note.youdao.com/s/C1javJgV)
1. **高峰时期流量增大**
2. 一个key存储了全部的任务内容，所有完成任务的都需要查询该key，当查询的量足够大时，对于存储这个key的节点就会占用较大资源（cpu，内存）, 这个时候应该拆分
3. 将该key 拆分成每个任务一个 key+任务id 的形式，任务的key会散落到redis实例下的不同节点，<font color="red">由多个节点分担这个key的查询压力，设置过期时间，让key在过期后会切换不同的节点，达到动态平衡的效果</font>
- [【有道云笔记】Redis缓存各种大key的的拆分方案](https://note.youdao.com/s/A9PZta3X)

## 删除单个大key
如果线上redis出现大key，断然不可立即执行del，因为大key的删除会造成阻塞。阻塞期间，所有请求都可能造成超时，当超时越来越多，新的请求不断进来，这样会造成redis连接池耗尽，尽而引发线上各种依赖redis的业务出现异常。
- **<font color="red">[【有道云笔记】redis缓存大key删除](https://note.youdao.com/s/M0bGM5MG)</font>**
   - 立即删除<font color="red">(各种scan查询，配合相关数据类型删除语法)</font>
   - 过期删
     - <font color="red">同步删除 过期策略（定期删除和惰性删除（惰性的淘汰策略）</font>
     - 异步删除 主动删除和程序被动删除

## keys与scan(scan查询命令代替各数据类型的keys)
- keys命令  遍历算法O(n) 没有limit
- 在正式的生产环境中一般不会直接使用keys这个命令，因为他会返回所有的键，如果键的数量很多会导致查询时间很长，进而导致服务器阻塞，所以需要scan来进行更细致的查找
- String O(1)
- List Hash Set Zset  O(n)  n越大。会阻塞主线程，如何del 先获取元素数量（5000），否则分批删除
- <font color="red">使用scan代替 O（N） 但是是分次进行 (不会阻塞线程) 能指定count  会返回游标（下次要执行）+ 数据 </font>
  - 根据返回的游标值是否为0 判断遍历结束
  - 缺点：遍历中途发生数据修改，之后能否遍历到不确定（根据游标来的 不是表面顺序），客户端要去重
``` go
scan cursor [MATCH pattern] [COUNT count] [TYPE type]  list    llen  lpop rpop
sscan key cursor [MATCH pattern] [COUNT count] set 代替smenbers
hscan key cursor [MATCH pattern] [COUNT count] hash 代替hgetall
zscan key cursor [MATCH pattern] [COUNT count] zset 代替zrange
```

### 参数demo
``` go
cursor表示游标，指查询开始的位置，count默认为1
MATCH可以采用模糊匹配找出自己想要查找的键
// list
127.0.0.1:6379[2]> scan 0 match mylist* count 20
1) "0"
2) 1) "mylist"
   1) "mylist2"
   2) "mylist1"
TYPE可以根据具体的结构类型来匹配该类型的键
127.0.0.1:6379[2]> scan 0 count 20 type list
1) "0"
2) 1) "mylist"
   1) "mylist2"
   2) "mylist1"

// set
127.0.0.1:6379[2]> sadd myset1 a b c d
(integer) 4
127.0.0.1:6379[2]> smembers myset1
1) "d"
2) "a"
3) "c"
4) "b"
127.0.0.1:6379[2]> sscan myset1 0
1) "0"
2) 1) "d"
   2) "c"
   3) "b"
   4) "a"
127.0.0.1:6379[2]> sscan myset1 0 match a
1) "0"
2) 1) "a"


// hash
127.0.0.1:6379[2]> hset myhset1 kk1 vv1 kk2 vv2 kk3 vv3
(integer) 3
127.0.0.1:6379[2]> hgetall myhset1
1) "kk1"
2) "vv1"
3) "kk2"
4) "vv2"
5) "kk3"
6) "vv3"
127.0.0.1:6379[2]> hscan myhset1 0
1) "0"
2) 1) "kk1"
   2) "vv1"
   3) "kk2"
   4) "vv2"
   5) "kk3"
   6) "vv3"

// zset
127.0.0.1:6379[2]> zadd myzadd1 1 zz1 2 zz2 3 zz3
(integer) 3
127.0.0.1:6379[2]> zrange myzadd1 0 -1 withscores
1) "zz1"
2) "1"
3) "zz2"
4) "2"
5) "zz3"
6) "3"
127.0.0.1:6379[2]> zscan myzadd1 0
1) "0"
2) 1) "zz1"
   2) "1"
   3) "zz2"
   4) "2"
   5) "zz3"
   6) "3"

```

## 估算需要多大的redis 
- http://www.redis.cn/redis_memory/
- MEMORY USAGE mykey命令，可以在测试环境查看这个key具体占用的字节，在根据我们目前的日活60w来提前预估可能会存的量，可以提前预估一下占用的内存

- 在 Redis 中，一个 key 占用的内存空间主要由如下三部分组成：
  1. 键名本身的长度。可以使用 STRLEN key 命令获取。
  2. 键值的长度。可以使用 STRLEN key 命令获取。
  3. Redis 内部的固定开销。
  - 公式：内存大小 = 键名长度 + 键值长度 + 固定开销

## 监控命令
监控操作 monitor 实时打印出 Redis 服务器接收到的命令，调试用。
![alt text](image3.png)


## aof持久化
![alt text](image4.png)

![alt text](image5.png)

- sentinel 阿里巴巴 分布式流控制神器  与三主三从的对比

- cluster 集群 与 codis

# php缓存相关

如果没有，就是使用文件缓存
1. PHP常用扩展（一） PHP字节码缓存——Opcache
https://www.zbpblog.com/blog-382.html
2. PHP常用扩展（二） PHP用户级缓存——APCu
https://blog.csdn.net/qq_33521184/article/details/129122207
3. 一文读懂 mmap 原理 零拷贝技术
https://juejin.cn/post/6956031662916534279
4. mmap原理 是什么 为什么 怎么用
https://juejin.cn/post/7006339007085248525