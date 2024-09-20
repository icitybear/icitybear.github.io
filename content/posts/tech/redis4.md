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

# 消息队列
- 如何基于Redis实现消息队列
  - 文章 https://zhuanlan.zhihu.com/p/651758438
  - 视频
  - 他趣的g2 tqmq 纯redis

- 单机cron

- Asynq分布式任务队列
{{< innerlink src="posts/tech/go38.md" >}}

- go 语言分布式消息队列 nsq
   - 文章 https://zhuanlan.zhihu.com/p/665893174
   - 视频

# 协程池
- 使用 Go Channel 构建可扩展的并发 worker pool
  - https://mp.weixin.qq.com/s/zmS5L-ZxHNYGo3SylibMWg
- Golang 协程池 Ants 实现原理 
  - 文章  https://zhuanlan.zhihu.com/p/597231892
  - 视频
- 基于协程池架构实现分布式定时器
  - 文章 [基于协程池架构实现分布式定时器](https://zhuanlan.zhihu.com/p/600380258)
  - 视频  
  
  


# 分布式事务
- [漫谈分布式事务实现原理]()
  - 文章 https://zhuanlan.zhihu.com/p/648556608 事务消息和 TCC 事务两种分布式事务实现方案的技术原理
- [从零到一搭建 TCC 分布式事务框架]()
  - 文章 https://zhuanlan.zhihu.com/p/650791238

- 本地事务表