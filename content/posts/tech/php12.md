---
title: "水平扩展 PHP 类功能-对象组合和trait" #标题
date: 2023-07-02T16:10:10+08:00 #创建时间
lastmod: 2023-07-02T16:10:10+08:00 #更新时间
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

面向接口编程，而不是具体的实现-可以减少代码对具体实现的依赖，降低系统耦合
优先使用对象组合而不是类继承的方式实现代码复用

# 对象组合
对象组合，简而言之，就是在一个类中组合（或者说依赖）另一个类而不是继承另一个类来扩展它的功能，如果说类继承是垂直（纵向）扩展类功能，那么对象组合则是水平（横向）扩展类功能

**[demo](https://laravelacademy.org/post/21645)**

# trait类
- 提示不能定义和 Trait 同名的属性
- Trait 和类相似，支持定义方法和属性，但不是类，不支持定义构造函数，因而不能实例化，只能被其他类使用，要在一个类中使用 Trait，可以通过 use 关键字引入，然后就可以在类方法中直接使用 trait 中定义的方法和属性
- 同一个 Trait 可以被多个类复用，从而突破 PHP 单继承机制的限制，有效提升代码复用性和扩展性。
- 我们在 Trait 中可以使用 $this 指向当前 Trait 定义的属性和方法，因为 Trait 最终会被类使用，$this 也就最终对应着被使用类的对象实例。在类中调用 Trait 属性，就好像调用自己的属性一样。
- 同名方法重写的优先级依次是：使用 Trait 的类 > Trait > 父类
- 引用多个 Trait 通过逗号分隔即可,指定使用多个 Trait 同名方法中的哪一个来替代其他的，这样会导致其他未选择方法被覆盖,仍然想调用其他 Trait 中的同名方法，PHP 还提供了别名方案
``` php
<?php
      use Power, Engine {
        Engine::printText insteadof Power;
        Power::printText as printPower;
        Engine::printText as printEngine;
    }
```
**[详细使用](https://note.youdao.com/s/9BZXGaNL)**
- Trait 还可以组合多个 Trait 构建更复杂的 Trait 实现更强大的功能,让每个 Trait 只完成一个功能（单一职责原则），然后通过 Trait 组合的方式灵活构建完成特定任务功能的 Trait