---
title: "epoll多路复用" #标题
date: 2024-08-20T10:50:18+08:00 #创建时间
lastmod: 2024-08-20T10:50:18+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- 网络编程
- 基础
- web技术
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
1. 23年 文章：[解析 Golang 网络 IO 模型之 EPOLL](https://zhuanlan.zhihu.com/p/609629545)
   - 视频：
2. 24年 文章：[万字解析 golang netpoll 底层原理](https://zhuanlan.zhihu.com/p/721422268)
   - 视频：

万字解析 golang netpoll 底层原理：这是一个力求温故而知新的重置篇章，希望在有了不同语言间横向对比的视角后，能够对 golang 底层 io 模型设计、方案取舍原因有着更加立体的认知. 在本文中，我们将涉及到的如下知识点：io多路复用概念、epoll实现原理、针对 golang 底层 epoll 应用细节以及 netpoll 框架模型进行源码级别的讲解.