---
title: "消息队列，分布式任务调度" #标题
date: 2024-09-19T18:01:22+08:00 #创建时间
lastmod: 2024-09-19T18:01:22+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- redis
- 消息队列
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
# MQ 
1. [日常工作，MQ的8种常用使用场景](https://mp.weixin.qq.com/s/6vDMvJ13RP2xPVCACBvW8w)
2. [RocketMQ 的设计理念](https://mp.weixin.qq.com/s/i_bwyEb8QqfVLxY-R9odag) 有点rpc思想

## 跳表
[基于golang从零到一实现跳表](https://mp.weixin.qq.com/s/fvfz6bdvsZJtGsdL0MPYoA)
- 由于 redis 采用单线程处理模型，因此不存在并发访问跳表的诉求

RocksDB 是一个高性能的嵌入式键值存储数据库，由 Facebook（现 Meta）基于 Google 的 LevelDB 开发并优化而来。
rocksdb 则采用多线程处理模型，并发读写内存数据结构时，需要兼顾数据一致以及操作性能，此时就需要使用到并发安全的跳表结构.
  - 后续我们在基于跳表构建需要支持并发操作的有序表时 [如何实现一个并发安全的跳表] https://mp.weixin.qq.com/s/7VhioGP007LDQnZ_w8GBBQ
## 时间轮
[基于 golang 从零到一实现时间轮算法](https://mp.weixin.qq.com/s/0HwuYTTe9cdT46advEZe0w)
- 单机版和 redis 分布式版

# 消息队列
## <font color="red">如何基于Redis实现消息队列</font>
  - 文章 https://zhuanlan.zhihu.com/p/651758438
  - 视频
  - tq企业的g2 tqmq 纯redis

## go 语言分布式消息队列 nsq （服务端客户端)
   - 文章 https://zhuanlan.zhihu.com/p/665893174
   - 视频


## Asynq分布式任务队列 （基于消息队列）
{{< innerlink src="posts/tech/go38.md" >}}

crontab 不支持集群；缺少监控报警、失败统计.
rocketMQ 延时队列	时间精度和灵活度不足.

# 理解定时任务cron
{{< innerlink src="posts/tech/go37.md" >}}
- tq企业基于kratos框架 
  - 进一步封装biz-cron-job包  https://git.internal.taqu.cn/go-modules/biz-cron-job
- https://github.com/shunfei/cronsun 基于cron分布式版本 后端：Go 语言 前端：Vue.js 存储：支持 etcd 或 MongoDB

# 分布式调度
- gojob开源项目 https://github.com/icitybear/gojob
  只依赖MySQL 数据库，分布式环境下使用内建的分布式协调机制, 不需要安装第三方分布式协调服务，如Zookeeper、Etcd等
  由于作业元数据保存在节点自己的存储引擎中，MySQL数据库只用来保存调度日志
# 协程池
- 使用 Go Channel 构建可扩展的并发 worker pool
  - https://mp.weixin.qq.com/s/zmS5L-ZxHNYGo3SylibMWg 
- <font color="red">Golang 协程池 Ants 实现原理 （进阶版）</font>
  - 文章  https://zhuanlan.zhihu.com/p/597231892
  - 视频

分布式定时器服务
# 基于协程池架构实现分布式定时器 (了解就行) 完整版就是workflow.timer
  - 文章 [基于协程池架构实现分布式定时器](https://zhuanlan.zhihu.com/p/600380258) xTimer
  - 视频 
  - 本质上还是单进程内的通信交互；
  - https://github.com/xiaoxuxiansheng/xtimer
  
  workflow.timer https://juejin.cn/post/7116320697139331103  
  workflow.timer 中，两个模块的交互方式选择了使用 MQ 异步通信的方式，并最终敲定使用 Apache Plusar 作为实际的消息组件，其中一个重要的原因就是希望使用到消息队列的 ACK 与消息重发机制.
  workflow.timer 基于 mq 实现模块串联，是真正意义上的解耦；xTimer 则基于协程池实现模块间的异步调用，本质上还是单进程内的通信交互；重新设计 xTimer 的核心目的就是降低接入成本，由于 MQ 组件得到抽离，xTimer 接入时只需要有常用的 mysql 和 redis 组件的支持即可，成本上更低.

# 分布式事务
- [漫谈分布式事务实现原理]()
  - 文章 https://zhuanlan.zhihu.com/p/648556608 事务消息和 TCC 事务两种分布式事务实现方案的技术原理
- [从零到一搭建 TCC 分布式事务框架]()
  - 文章 https://zhuanlan.zhihu.com/p/650791238

- 本地事务表