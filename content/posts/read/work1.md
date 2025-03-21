---
title: "理解需求" #标题
date: 2025-02-13T18:39:09+08:00 #创建时间
lastmod: 2025-02-13T18:39:09+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- read
tags: # 标签
- 沟通
- 工作
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

主要应用5W2H模型进行需求理解：
从需求的整体和局部两个方面了解：
整体：指对用户提供的整个服务的需求： 例如：用户所看到的帖子推荐
局部：指各个角色或部门： 例如：大数据需要提供的一些人货场的相关数据；数据平台提供AB实验能力支持
# 整体：  有助于我们了解需求的全貌
1. What   想要做什么？目的：
2. Why  了解需求背景 （有些背景是很敷衍的， 比如：因为想看什么什么数据， 那为什么要看这个数据？这个数据对目标【what】有什么帮助？为什么是这个指标？）
  1. 为什么？
  2. 为什么？
  3. 为什么？
3. How  了解需求目的的实现路径
  1. 想想有没有更好的路径？
  2. 理解需求：多问问为什么？ 
  3. 需求拆解，有哪些角色一起支持这个需求？每个角色做什么支持？（了解全局，如果让你own这个需求，你会怎么设计或安排？跟当前的设计和安排有什么出入？）
4. When  各个节点的时间节奏？
5. Who  各个角色分别是谁？是否存在依赖？依赖方时间节点是否OK？哪方面的问题需要找谁支持？
6. Where  
7. How much  需要多少资源（人力，物力，时间，是否值得？如果成本太高，是否有B方案？）
# 局部：有助于提升我们所提供的服务质量
1. What      我们自己需要提供哪些支持？输入是什么？输出是什么？ 
    1. 需求边界包含哪些？不包含哪些？
    2. 主流程是什么？ 有哪些步骤？每个步骤有哪些异常点？
    3. 异常流程有哪些？
    4. 有什么特殊要求？时效性、 QPS、响应时长、稳定性？
    5. 需求确认，把你理解的跟需求方确认一遍
2. Why  同整体
3. When   内部每个步骤的时间节点
4. Who   内部是否有其他依赖？依赖谁？时间节点是怎样？
5. Where 
6. How  
    1. 内部的实现路径？
    2. 能否满足what要求？
    3. 如果无法满足？B计划是啥？应该找谁求助？