---
title: "PHP堆,栈,数据段,代码段" #标题
date: 2023-06-24T20:22:15+08:00 #创建时间
lastmod: 2023-06-24T20:22:15+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- php
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

对象在PHP里面和整型、浮点型一样，也是一种数据类，都是存储不同类型数据用的
<font color="red">在运行的时候都要加载到内存中去用，那么对象在内存里面是怎么体现的呢？</font>

# 内存逻辑上分
内存从逻辑上说大体上是分为4段，栈空间段、堆空间段、代码段、初始化静态段，程序里面不同的声明放在不同的内存段里面。
1. 数据段（data segment）通常是指用来存放<font color="red">程序中已初始化且不为0的全局变量</font>如：静态变量和常量
2. 代码段（code segment / text segment）通常是指用来存放程序执行代码的一块内存区域，比如函数和方法
3. 栈空间段是<font color="red">存储占用相同空间长度并且占用空间小的数据类型</font>的地方，比如说整型1，10，100，1000，10000，100000 等等，在内存里面占用空间是等长的，都是64 位4 个字节。
4. 堆：<font color="red">数据长度不定长，而且占有空间很大的数据类型的数据</font>放在那内存的放在堆内存里面的。

# 总结
- 数据段:字符串常量 全局变量 静态变量
- 代码段:函数 运行的代码
- 栈:基本数据类型、 局部变量、类的引用(指向堆空间段)
- 堆:new出来的对象
