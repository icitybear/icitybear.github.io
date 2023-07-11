---
title: "php静态属性和静态方法" #标题
date: 2023-07-02T17:55:42+08:00 #创建时间
lastmod: 2023-07-02T17:55:42+08:00 #更新时间
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
# 静态属性和方法概述
- 通过 static 关键字来修饰静态属性和方法
- 直接通过类引用，所以又被称作类属性和类方法（相应的，非静态属性和非静态方法需要实例化后通过对象引用，因此被称作对象属性和对象方法）
- 静态属性和方法可以通过 类名::属性/方法 的方式调用,在类内部方法中，需要通过 self:: 引用当前类的静态属性和方法
- 没有实例化的情况下，$this 指针指向的是空对象
- 作用域是整个类，而不是某个对象,支持动态修改
- 支持设置 private、protected、public 三种可见性级别
- 静态属性和方法也可以被子类继承，静态方法还可以被子类重写,通过 __CLASS__ 可以获取当前类的类名
- 和 $this 指针始终指向持有它的引用对象不同，self 指向的是定义时持有它的类而不是调用时的, 所以出现后期静态绑定的特性

# 后期静态绑定的特性 static::
- 针对静态方法的调用，通过 static::调用，如果是在定义它的类中调用，则指向当前类，此时和 self 功能一样，如果是在子类或者其他类中调用，则指向调用该方法所在的类
- 通过 static::class 来指向当前调用类的类名，例如我们可以通过它来替代 __CLASS__

# 魔术方法
- 魔术方法以 __ 开头，这是一类特殊的系统方法，因此不要在自定义方法名中添加 __ 前缀 
__construct()、__destruct()、__call()、__callStatic()、__get()、__set()、__isset()、__unset()、__sleep()、 __wakeup()、__toString()、__invoke()、__set_state()、__clone() 和 __debugInfo()。
- [魔术方法和对象的序列化](https://laravelacademy.org/post/21651)