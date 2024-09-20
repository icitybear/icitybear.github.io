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
  - lua
  - 文章 https://zhuanlan.zhihu.com/p/629247043