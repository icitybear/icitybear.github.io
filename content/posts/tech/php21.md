---
title: "php发展历史" #标题
date: 2024-05-06T14:29:09+08:00 #创建时间
lastmod: 2024-05-06T14:29:09+08:00 #更新时间
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

- 1997 personal home page

- 2000 php4 zend引擎 c

- 2001 smarty模版引擎, dedecms织梦 内容管理系统

- 2003 wordpress 博客 应用管理系统

- 2004 php5 zend2, facebook

- 2005 symfony框架,  cakePhp框架

- 2006 ci框架, zend frame框架

- 2008 yii框架

- 2009 thinkPhp2框架

- 2010 <font color="red">php5.3 命名空间,  php-fpm进程管理器</font>

- 2011 
  - laravel qps低 , 
  - facebook (c++ HHVM) 类似JVM hack超集（加类型标注）
  - php扩展 swoole 非阻塞io模型和事件驱动的编程方式 支持多进程和协程

- 2012 
  - <font color="red">php包管理工具 composer</font>
  - php5.5 OPcache扩展（缓存中间代码）
  - thinkPhp3框架

- 2013 <font color="red">workerman框架（使用非阻塞io模型和事件驱动），性能比go的gin还高</font>
- 
- 2014 小程序微商

- 2015 
  - php7 类型标注  
  - <font color="red">lumen(laravel)微服务</font>
  
- 2016 <font color="red">thinkPhp5框架</font>

- 2018 
  - RoadRunner使用。go开发的高性能应用服务，进程管理器 替代php-fpm，利用协程和事件驱动模型与php解释器进行高效的通信
  - livewire（larvel社区开发的php库，类似前端vue）

- 2019 <font color="red">hyperf框架基于swoole</font>

- 2020 php8 jit编译器, 以及很多语言特性

- 2021 <font color="red">larvel Octane加速器</font>, 使用swoole roadrunner进行加速 不需要改动代码