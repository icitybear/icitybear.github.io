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

3. [万字学习笔记：cloudwego/netpoll](https://zhuanlan.zhihu.com/p/1896654310688933137)
4. [万字学习笔记：cloudwego/kitex](https://zhuanlan.zhihu.com/p/1902680493620700799)
   kitex 是 go 实现的高性能的 rpc 框架，集成了编解码 codec、通信 transport、服务发现 discovery、负载均衡 loadbalance、链路追踪 trace 等一系列模块，通过预留接口的方式保留了灵活的扩展度.
   
万字解析 golang netpoll 底层原理：将涉及到的如下知识点：io多路复用概念、epoll实现原理、针对 golang 底层 epoll 应用细节以及 netpoll 框架模型进行源码级别的讲解.
在设计 io 模型时，golang 采用了 linux 系统提供的 epoll 多路复用技术，然而为了因为 epoll_wait 操作而引起 m（thread）粒度的阻塞，golang 专门设计一套 netpoll 机制，使用用户态的 gopark 指令实现阻塞操作，使用非阻塞 epoll_wait 结合用户态的 goready 指令实现唤醒操作，从而将 io 行为也控制在 g 粒度，很好地契合了 gmp 调度体系.