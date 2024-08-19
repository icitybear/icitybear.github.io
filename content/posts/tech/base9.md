---
title: "访问权限控制casbin" #标题
date: 2024-08-19T11:45:22+08:00 #创建时间
lastmod: 2024-08-19T11:45:22+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- go包
- 访问权限
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

# 简介
Casbin是一个强大的、高效的开源访问控制框架，其权限管理机制支持多种访问控制模型。

# 文档
[文档地址-使用说明](https://www.kancloud.cn/oldlei/casbin/1289451)

# b站视频
[访问控制casbin扫盲教程](https://www.bilibili.com/video/BV18bePeuEBu)

# demo
创建一个Casbin决策器需要有一个模型文件和策略文件为参数
``` go
go get github.com/casbin/casbin/v2

import "github.com/casbin/casbin"

e := casbin.NewEnforcer("path/to/model.conf", "path/to/policy.csv")
```