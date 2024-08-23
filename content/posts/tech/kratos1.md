---
title: "Kratos" #标题
date: 2024-08-23T11:17:43+08:00 #创建时间
lastmod: 2024-08-23T11:17:43+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- 
tags: # 标签
-
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
# kratos框架
** 后续自己整理  https://pandaychen.github.io/tags/#Kratos ** 很多知识点可以阅读

- 限流器
  - bbr算法实现
- 熔断器
- tracking
- 指标监控和普罗米修斯
- errgroup，包含原生和封装
  - https://pandaychen.github.io/2020/07/20/KRATOS-ERRGROUP/
- wardon


# Kratos 示例集
- https://github.com/go-kratos/examples/blob/main/README-CN.md

[English](README.md) | 简体中文

| 项目名                          | 说明                               |
|------------------------------|----------------------------------|
| [auth/jwt](./auth/jwt)       | JWT使用示例                          |
| [blog](./blog)               | 博客系统，简单的CRUD示例                   |
| [casbin](./casbin)           | JWT认证和Casbin鉴权的示例                |
| [chatroom](./chatroom)       | 最简单的Websocket聊天室示例               |
| [config](./config)           | 文件配置和远程配置中心使用示例                  |
| [cqrs](./cqrs)               | CQRS模式实现示例，主要是Kafka的使用。          |
| [errors](./errors)           | 错误处理示例                           |
| [event](./event)             | 使用Kafka等MQ收发消息示例                 |
| [header](./header)           | 自定义HTTP的Header示例                 |
| [helloworld](./helloworld)   | RPC调用示例                          |
| [http](./http)               | HTTP相关应用的示例，如：CORS、上传文件等。        |
| [i18n](./i18n)               | 国际化与本地化的示例                       |
| [log](./log)                 | 日志使用示例                           |
| [metadata](./metadata)       | 元数据使用示例                          |
| [metrics](./metrics)         | 度量系统使用示例                         |
| [middleware](./middleware)   | 中间件使用示例                          |
| [realtimemap](./realtimemap) | 实时公交地图，主要是MQTT、Websocket、RPC综合使用 |
| [registry](./registry)       | 注册中心使用示例                         |
| [selector](./selector)       | 节点选择器使用示例                        |
| [stream](./stream)           |                                  |
| [swagger](./swagger)         | swagger api使用示例                  |
| [tls](./tls)                 | TLS证书使用示例                        |
| [traces](./traces)           | 链路追踪系统使用示例                       |
| [transaction](./transaction) | 数据库事务使用示例                        |
| [validate](./validate)       | Protobuf参数校验器使用示例                |
| [ws](./ws)                   | 最简单的Websocket示例                  |