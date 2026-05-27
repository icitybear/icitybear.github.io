---
title: "k/v一致性hash, hash环的演进" #标题
date: 2024-08-20T10:47:47+08:00 #创建时间
lastmod: 2024-08-20T10:47:47+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- redis
- 缓存
- go包
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
# Redis是单线程模型
Redis6.0引入多线程IO，但多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程。默认是不开启的，需要进程启动前开启配置，并且在运行期间无法通过 config set 命令动态修改。

- https://segmentfault.com/a/1190000046792622

# redis常用集群以及性能
https://segmentfault.com/a/1190000042301957

1. Codis 采用 Pre-sharding 的技术来实现数据的分片, 默认分成 1024 个 slots (0-1023), 对于每个Key来说, 通过以下公式确定所属的 Slot Id : SlotId = crc32(key) % 1024。
  - Codis 是 Wandoujia Infrastructure Team 开发的一个分布式 Redis 服务
  - 一个无限内存的 Redis 服务, 有动态扩/缩容的能力. 对偏存储型的业务更实用. Codis 是不支持 SUBPUB 之类的指令

2. Redis cluster 默认分配了 16384 个slot，当我们set一个key 时，会用CRC16算法来取模得到所属的slot，然后将这个Key 分到哈希槽区间的节点上，具体算法就是：CRC16(key) % 16384
  - Redis 集群的优势:1.自动分割数据到不同的节点上。2.整个集群的部分节点失败或者不可达的情况下能够继续处理命令。Redis集群并不支持处理多个Keys的命令
  - 类似一致性hash

# <font color="red">一致性hash</font>
1. [一致性哈希算法原理解析](https://zhuanlan.zhihu.com/p/653210271)
    - b站视频 https://www.bilibili.com/video/BV1fF41127pg
3. [从零到一落地实现一致性哈希算法](https://zhuanlan.zhihu.com/p/654778311)
	- b站视频 https://www.bilibili.com/video/BV1F94y1a7fA

git项目地址: http://github.com/xiaoxuxiansheng/consistent_hash


- 美团大规模KV存储挑战与架构实践-b站视频
  - 4399缓存一致性hash
  - https://blog.csdn.net/zhaozhiqiang1981/article/details/139564621


# <font color="red">分布式锁</font> 重要
- [Golang分布式锁技术攻略](https://www.bilibili.com/video/BV1Pm4y1b76u)
  - 文章 https://zhuanlan.zhihu.com/p/626924850
- etcd实现分布式锁
  - paxos协议(raft的前身) b站视频
  - [文解析raft算法原理](https://zhuanlan.zhihu.com/p/600147978)
  - Raft [解析分布式共识算法之Raft算法](https://www.bilibili.com/video/BV1Kz4y1H7gw)

- redis实现[Redis分布式锁进阶篇-b站视频](https://www.bilibili.com/video/BV1wP411X7sc)
  - 文章 https://zhuanlan.zhihu.com/p/629247043
  - 有封装的包
  - 简单版（redis的lua脚本）


## redis的lua脚本 简单版

- <font color="red">单节点 Redis 锁，核心用两段 Lua 脚本，所以Redis 主从切换可能丢锁（极端情况） 比如go语言的sync.Mutex直接是单机锁</font>
- 锁的用途是防重复执行（幂等保护），而非强一致性互斥

``` go
package redislock

import (
	"context"
	"math/rand"
	"strconv"
	"sync/atomic"
	"time"

	"github.com/go-kratos/kratos/v2/log"
	"github.com/google/uuid"
	"github.com/redis/go-redis/v9" // 使用的redis客户端
)

const (
	tolerance       = 500 // milliseconds
	millisPerSecond = 1000
	lockCommand     = `if redis.call("GET", KEYS[1]) == ARGV[1] then
    return redis.call("SET", KEYS[1], ARGV[1], "PX", ARGV[2])
end
return redis.call("SET", KEYS[1], ARGV[1], "NX", "PX", ARGV[2])`
	delCommand = `return redis.call("GET", KEYS[1]) == ARGV[1] and redis.call("DEL", KEYS[1]) or 0`
)

// A RedisLock is a redis lock.
type RedisLock struct {
	redis   *redis.Client
	seconds uint32
	key     string
	id      string
}

func init() {
	rand.Seed(time.Now().UnixNano())
}

// NewRedisLock returns a RedisLock.
func NewRedisLock(redis *redis.Client, key string) *RedisLock {
	return &RedisLock{
		redis: redis,
		key:   key,
		id:    uuid.New().String(),
	}
}

// Acquire acquires the lock.
func (rl *RedisLock) Acquire() (bool, error) {
	return rl.AcquireCtx(context.Background())
}

// AcquireCtx acquires the lock with the given ctx.
func (rl *RedisLock) AcquireCtx(ctx context.Context) (bool, error) {
	seconds := atomic.LoadUint32(&rl.seconds)
	resp := rl.redis.Eval(ctx, lockCommand, []string{rl.key}, []string{
		rl.id, strconv.Itoa(int(seconds)*millisPerSecond + tolerance),
	})
	if resp.Err() == redis.Nil {
		return false, nil
	} else if resp.Err() != nil {
		log.Errorf("Error on acquiring lock for %s, %s", rl.key, resp.Err())
		return false, resp.Err()
	} else if resp == nil {
		return false, nil
	}

	if resp.Val() == "OK" {
		return true, nil
	}

	log.Errorf("Unknown reply when acquiring lock for %s: %v", rl.key, resp)
	return false, nil
}

// Release releases the lock.
func (rl *RedisLock) Release() (bool, error) {
	return rl.ReleaseCtx(context.Background())
}

// ReleaseCtx releases the lock with the given ctx.
func (rl *RedisLock) ReleaseCtx(ctx context.Context) (bool, error) {
	resp := rl.redis.Eval(ctx, delCommand, []string{rl.key}, []string{rl.id})
	if resp.Err() != nil {
		return false, resp.Err()
	}

	reply, err := resp.Int64()
	if err != nil {
		return false, err
	}

	return reply == 1, nil
}

// SetExpire sets the expiration.
func (rl *RedisLock) SetExpire(seconds int) {
	atomic.StoreUint32(&rl.seconds, uint32(seconds))
}
```
### 封装成看门狗
``` go
package data

import (
	"context"
	"git.internal.xxx.cn/go-modules/kratos-extends/redislock"
	"github.com/go-kratos/kratos/v2/log"
)

type WatchDogLock struct {
	key  string
	lock *redislock.RedisLock
}

func NewWatchDogLock(key string) *WatchDogLock {
	lock := redislock.NewRedisLock(dataIns.rdb, key)
	lock.SetExpire(10)

	return &WatchDogLock{
		key:  key,
		lock: lock,
	}
}

func NewWatchDogLockWithExpire(key string, expire int) *WatchDogLock {
	lock := redislock.NewRedisLock(dataIns.rdb, key)
	lock.SetExpire(expire)

	return &WatchDogLock{
		key:  key,
		lock: lock,
	}
}

func (w *WatchDogLock) setLockExpire(expire int) {
	w.lock.SetExpire(expire)
}

func (w *WatchDogLock) AcquireCtx(ctx context.Context) bool {
	ok, err := w.lock.Acquire()
	if err != nil {
		log.Context(ctx).Errorf("WatchDogLock  key:%s Acquire error:%+v", w.key, err)
	}
	return ok
}

func (w *WatchDogLock) Acquire() bool {
	ok, err := w.lock.Acquire()
	if err != nil {
		log.Errorf("WatchDogLock key:%s Acquire error:%+v", w.key, err)
	}
	return ok
}

func (w *WatchDogLock) Release() bool {
	ok, err := w.lock.Release()
	if err != nil {
		log.Errorf("WatchDogLock key:%s Release error:%+v", w.key, err)
	}
	return ok
}

func (w *WatchDogLock) ReleaseCtx(ctx context.Context) bool {
	ok, err := w.lock.ReleaseCtx(ctx)
	if err != nil {
		log.Context(ctx).Errorf("WatchDogLock  key:%s ReleaseCtx error:%+v", w.key, err)
	}
	return ok
}


```
- 使用
``` go
	watchDog := data.NewWatchDogLockWithExpire("tt_auto_refresh_token_lock", 60)
	if !watchDog.AcquireCtx(ctx) {
		log.Context(ctx).Warnf("[TtTokenRefreshJob]执行锁被抢占")
		return
	}
	defer watchDog.ReleaseCtx(ctx)
```
1. 用 UUID 作为持有者标识，保证只有持锁方能释放
2. 支持"伪重入"——同一持有者再次加锁时刷新过期时间
3. 过期时间 = 设定秒数 × 1000 + 500ms 容差
4. <font color="red">没有自动续期（watchdog）机制，项目层 WatchDogLock 名字有误导性，实际只是封装了 acquire/release</font> 

## redis的setNx+lua （额外写法）

![alt text](image1.jpg)
这里几个注意点
1. 设置redis值的时候使用 NX EX 命令参数，一次命令搞定
2. 释放锁的时候，ctx使用的是 context.Background()，避免传入的 ctx 过期导致锁释放失败
3. 释放锁的时候 检查 锁内容，避免误删其他请求加的锁，比如设置5秒超时，执行其他操作耗时6秒，6秒后去删除锁，不检查锁内容的话，这个时候删除的就是其他请求加的锁了
4. redis 设置超时时间允许使用毫秒级，这里 val 使用微秒级，避免传递毫秒级 ttl 可能造成的误删
5. 使用 ttl ...time.Duration 来实现 默认值操作，这里的 ttl 是 time.Duration 数组，数组为0用默认值，不为0取第一个值 

- <font color="red">释放锁的逻辑优化一下</font>
![alt text](image2.jpg)
1. 使用协程执行，避免 redis 网络抖动导致释放锁阻塞 接口返回
2. 为了避免 网络抖动导致 协程 执行时间过长，在请求 redis 的 context 上加了 timeout
3. 为了让日志可以获取到请求上下文，需要继承请求 ctx 数据，但不能继承其过期时间，实现 WithoutCancel函数（go 1.21.0 已内置）

# <font color="red">Redsync 红锁(redis)</font>
- [redis官方包提供] https://github.com/go-redsync/redsync

|维度	|简单版	|Redsync|
|---------|---------|------------|
|节点模型	|单 Redis 实例	|多实例 Redlock（也可单实例）|
|容错性	|Redis |主从切换可能丢锁	|多数派存活即可|
|重试机制	|无，调用方自行处理	|内置可配|
|续期/watchdog	|无	|有 Extend()|
复杂度	|极简（~100行）	|较重（依赖多）|
|适用场景	|单 Redis、短时互斥、容忍极端情况丢锁	|高可靠性要求、多节点部署|


- 支持多节点（N 个独立 Redis 实例，多数派加锁成功才算获取）
- 同样用 Lua 脚本保证原子性
- 内置 retry 策略（可配重试次数、间隔、抖动）
- 支持 Extend() 续期
- 锁的有效时间 = TTL - 获取锁消耗的时间（时钟漂移补偿）

<font color="red">业务对"绝对不能重复执行"有强要求（比如扣款），单节点锁不够，需要 Redsync + 多独立 Redis 实例，或者用数据库行锁/乐观锁做兜底</font>

## 封装看门狗

1. 过期时间

   使用分布式锁很重要的问题，那就是一定要设置一个过期时间，这样才能保证即使拿到锁的进程挂掉了，只要锁的过期时间已到，锁也一定会被自动释放掉 (redsync 内部设置了默认值 8s)
2. 锁自动续期

   分布式锁一定要设置一个过期时间了，但是这会带来另外一个问题：如果我们的业务代码还没执行完，锁就过期自动释放了，那么此时另外一个进程成功拿到这把锁，也来访问竞态资源，那分布式锁不就失去意义, 所以需要自动续期

- 看门狗

通过为分布式锁设置过期时间，再配合子 goroutine 自动续期的功能，我们就能保证，持有锁的进程挂掉时不会影响其他进程获取锁，并且还能实现业务执行完成后才释放锁。而这个实现分布式锁自动续期的程序，我们通常把它叫做“看门狗”。

关于分布式锁的续期时常和间隔周期的问题，一般来说，续期的时间可以设置为等于过期时间，即锁的过期时间设为 5 秒，那么每次也只续期 5 秒，redsync 内部也是这么做的，至于间隔多久续期一次，这个时间肯定是要小于过期时间 5 秒的，通常设为锁过期时间的 1/3 或 1/2 都可以

``` go
package main

import (
    "context"
    "log/slog"
    "time"

    "github.com/go-redsync/redsync/v4"                  // 引入 redsync 库，用于实现基于 Redis 的分布式锁
    "github.com/go-redsync/redsync/v4/redis/goredis/v9"// 引入 redsync 的 goredis 连接池
    goredislib "github.com/redis/go-redis/v9"           // 引入 go-redis 库，用于与 Redis 服务器通信
)

func main() {
    // 创建一个 Redis 客户端
    client := goredislib.NewClient(&goredislib.Options{
        Addr:     "localhost:36379", // Redis 服务器地址
        Password: "nightwatch",
    })

    // 使用 go-redis 客户端创建一个 redsync 连接池
    pool := goredis.NewPool(client)

    // 创建一个 redsync 实例，用于管理分布式锁
    rs := redsync.New(pool)

    // 创建一个名为 "test-redsync" 的互斥锁（Mutex） 默认内置8秒过期
    mutex := rs.NewMutex("test-redsync", redsync.WithExpiry(5*time.Second))

    // 创建一个上下文（context），一般用于控制锁的超时和取消
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    // 获取锁，如果获取失败（例如锁已被其他进程持有），会返回错误
    if err := mutex.LockContext(ctx); err != nil {
        panic(err) // 如果获取锁失败，程序会 panic
    }

    // 看门狗，实现锁自动续约
    stopCh := make(chanstruct{})
    ticker := time.NewTicker(2 * time.Second) // 每隔 2s 续约一次
    defer ticker.Stop()
    gofunc() {
        for {
            select {
            case <-ticker.C:
                // 续约，延长锁的过期时间
                if ok, err := mutex.ExtendContext(ctx); !ok || err != nil {
                    slog.Error("Failed to extend mutex", "err", err, "status", ok)
                } else {
                    slog.Info("Successfully extend mutex")
                }
            case <-stopCh:
                slog.Info("Exiting mutex watchdog")
                return
            }
        }
    }()

    // 执行业务逻辑
    time.Sleep(6 * time.Second)

    // 通知看门狗停止自动续期
    stopCh <- struct{}{}

    // 释放锁，如果释放失败（例如锁已过期或不属于当前进程），会返回错误
    if _, err := mutex.UnlockContext(ctx); err != nil {
        panic(err) // 如果释放锁失败，程序会 panic
    }
}
```



# etcd实现的分布式锁
- 一般使用redis
- etcd特性：会话型锁的设计需要客户端主动维护生命周期 <font color="red"etcd的concurrency包</font>

排查etcd应用分布式锁而导致的泄露与死锁问题
1. 使用go tool pprof观察goroutine增长趋势
2. 现象描述
   服务出现数据入库失败，且伴随内存持续增长。关键现象：
   1. 锁残留：通过`etcdctl get --prefix /my-lock/`可看到锁KEY长期存在
   2. 租约续期：`etcdctl lease timetolive`显示租约TTL不断重置
   3. 资源增长：Go程序协程数随请求量线性增长（可通过pprof观测）
3. 简单复现
``` go
func main() {
    // ... etcd client初始化代码不变...

    // 关键问题点1：缺失session关闭
    session, _ := concurrency.NewSession(cli)
    // defer session.Close() // 故意注释导致协程泄漏

    // 关键问题点2：未释放锁
    mutex := concurrency.NewMutex(session, "/my-lock/")
    ctx, _ := context.WithTimeout(context.Background(), 5*time.Second)
    _ = mutex.Lock(ctx)
    
    // ...业务逻辑代码...
    
    // 关键问题点3：阻止程序退出（仅用于demo）
    select {} 
}
```
1. 原因分析
``` go
+--------------+      定期续约      +------+
|  Go routine  |----------------->| etcd |
+--------------+  (KeepAlive)     +------+
       ▲
       │ 未调用Close()
       └──────+
              |
+---------------------------+
| session.Close() 核心作用：|
| 1. 停止续约协程           |
| 2. 释放租约              |
+---------------------------+
```
   - 协程泄漏路径： NewSession() → go keepAlive协程 → client.KeepAlive() → 后台协程续约(sendKeepAliveLoop)
   - 资源释放路径： session.Close() → Orphan() → 关闭上下文 → 触发keepAlive协程退出
1. 修复（最佳实践）
资源释放三原则：
• 对每个NewSession()必须配对defer Close()

• 锁操作必须包裹在Lock()/Unlock()中

• 使用带超时的上下文（建议不超过5s）
``` go
func main() {
    // ...初始化代码不变...

    session, err := concurrency.NewSession(cli)
    if err != nil {
        log.Fatal(err)
    }
    defer session.Close() // 新增关键修复

    mutex := concurrency.NewMutex(session, "/my-lock/")
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel() // 确保上下文取消

    if err := mutex.Lock(ctx); err != nil {
        log.Fatal(err)
    }
    defer mutex.Unlock(context.TODO()) // 双保险释放锁

    // ...业务逻辑...
}
```