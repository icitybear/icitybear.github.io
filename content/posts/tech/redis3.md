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

# 一致性hash
1. [一致性哈希算法原理解析](https://zhuanlan.zhihu.com/p/653210271)
- 对应b站视频
2. [从零到一落地实现一致性哈希算法](https://zhuanlan.zhihu.com/p/654778311)

- <font color="red">[美团大规模KV存储挑战与架构实践-b站视频](https://b23.tv/CabuXug)</font>
  - 4399缓存一致性hash
  - https://blog.csdn.net/zhaozhiqiang1981/article/details/139564621

# 分布式锁
- [Golang分布式锁技术攻略](https://b23.tv/iVOODdQ)
  - 文章 https://zhuanlan.zhihu.com/p/626924850
- etcd实现分布式锁
  - raft
- redis实现（一般是这个）[Redis分布式锁进阶篇-b站视频](https://b23.tv/aRUuFZQ)
  - 文章 https://zhuanlan.zhihu.com/p/629247043
  - 有封装的包
  - 简单版（redis的lua脚本）
## redis的lua脚本
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
- 封装成看门狗
``` go
package data

import (
	"context"
	"git.internal.taqu.cn/go-modules/kratos-extends/redislock"
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
## redis的setNx+lua

![alt text](image1.jpg)
这里几个注意点
1. 设置redis值的时候使用 NX EX 命令参数，一次命令搞定
2. 释放锁的时候，ctx使用的是 context.Background()，避免传入的 ctx 过期导致锁释放失败
3. 释放锁的时候 检查 锁内容，避免误删其他请求加的锁，比如设置5秒超时，执行其他操作耗时6秒，6秒后去删除锁，不检查锁内容的话，这个时候删除的就是其他请求加的锁了
4. redis 设置超时时间允许使用毫秒级，这里 val 使用微秒级，避免传递毫秒级 ttl 可能造成的误删
5. 使用 ttl ...time.Duration 来实现 默认值操作，这里的 ttl 是 time.Duration 数组，数组为0用默认值，不为0取第一个值 

- 释放锁的逻辑优化了下
![alt text](image2.jpg)
1. 使用协程执行，避免 redis 网络抖动导致释放锁阻塞 接口返回
2. 为了避免 网络抖动导致 协程 执行时间过长，在请求 redis 的 context 上加了 timeout
3. 为了让 日志可以获取到请求上下文，需要继承 请求 ctx 数据，但不能继承其过期时间，实现 WithoutCancel 函数（go 1.21.0 已内置）