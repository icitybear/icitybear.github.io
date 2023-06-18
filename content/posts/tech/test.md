---
title: "Test" #标题
date: 2023-06-15T11:08:38+08:00 #创建时间
lastmod: 2023-06-15T11:08:38+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- 
tags: # 标签
- demo
keywords: 
- 
description: "" #描述 每个文章内容前面的展示描述
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true #是否展示评论
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: "posts/tech/test/go2.png" #图片路径：posts/tech/文章1/picture.png
    caption: "测试封面图" #图片底部描述
    alt: ""
    relative: false

# reward: true # 打赏
mermaid: true #是否开启mermaid
---
## 图片
文章内容的直接图片
![](go2.png)

## 目录1
普通问题

## 目录2

<div align="center">

{{<mermaid>}}
    flowchart LR
        A --> B & C --> D
{{</mermaid>}}   

</div>

## bilibili视频
{{< bilibili BV1xW4y1a7NK >}} 

## 自己博客内链接
{{< innerlink src="posts/tech/test1.md" >}}  
