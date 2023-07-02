---
title: "面向对象" #标题
date: 2023-07-02T15:02:37+08:00 #创建时间
lastmod: 2023-07-02T15:02:37+08:00 #更新时间
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

# 概述
- 类是对象的抽象模板，而对象是类的具体实例
- 在面向对象编程中，对象是程序的基本单元，一个对象包含了数据和操作数据的函数。
- 对象包含的数据称之为类属性（Property），操作数据的函数称之为类方法（Method）。
  - 类属性，通过 var 来定义变量属性(php4前，后面省略了)，通过 const 来定义常量属性
  - 类方法，修饰符
- class 进行声明类,PHP 内置的 class_exists 方法判断该类是否存在
- 通过 new 关键字进行类的实例化,<font color="red">对象级别</font>的属性和方法，都是通过箭头符 -> 进行访问的,类级别的（类常量 静态常量,方法等，通过 类名::xxx或者self::）

- $this 变量，它指向的是<font color="red">当前对象实例引用</font>，可以用于在类内部调用对象级别属性和方法(类级别用 self:: 访问)
  
# 访问控制
PHP 通过 public（公开）、protected（保护）、private（私有）**关键字**控制类属性和方法的可见性：

- 对于通过 public 声明的属性和方法，在类以外和继承类中均可见；
- 对于通过 protected 声明的属性和方法，仅在继承类（支持多个层级）中可见，在类以外不可见；
- 对于通过 private 声明的属性和方法，<font color="red">仅在当前类内部可见，在继承类和类之外不可见。</font>

**依然可以通过反射的方式将 protected 或者 private 级别的属性和方法变成类以外可以访问**

# 继承
继承通过 extends 关键字实现, 单继承机制
# 封装
调用者无需关心对象方法实现细节, 合理的设置属性和方法的可见性。
# 多态
## 重写：当一个父类和子类有一个方法，参数和名字完全一致， //子类重写父类的方法，实现不同的功能 这叫做多态
那么子类方法会覆盖父类的方法。要求参数也必须一致。（父类和子类之间）
## 重载：函数重载指方法的名称相同而参数形式不同，且能够通过函数的参数个数或参数类型将这些同名的函数区分开来，
调用不发生混淆。即当调用的时候，<font color="red">虽然方法名字相同，但根据参数的不同</font>可以自动调用相应的函数。
使用__set(),__get()对属性进行调用，使用__call()对方法进行重载。（一个类之间）
## 类作为参数类型声明，类型转换
由于子类一定包含了父类的公开方法，所以当类作为参数类型声明时，如果声明类型为父类，则可以传入子类对象，反过来，如果声明类型为子类，则不能传入父类对象。会错误提示时不能将父类对象转化为子类对象，因为存在方法不兼容

# 抽象类与接口
实际面向对象编程实践中，并不推荐使用具体的类作为类型声明，因为当我们在声明这个类型约束时，更多考虑的是可以在对应方法中调用这个类型提供的某些方法，然后在调用该方法的地方传入的对象实例只要实现了这些方法即可，这样，<font color="red">该方法就不会和具体的类绑定，从而提高了代码的扩展性和复用性。</font>

## 抽象类
- 抽象方法是通过 abstract 关键字修饰的方法，抽象方法只是一个方法声明，不能有具体实现,可见性不做限制
- 只要某个类包含了至少一个抽象方法，它就是抽象类，抽象类也需要通过 abstract 关键字修饰
- 抽象类本身不能被实例化，只能被子类继承extends，继承了抽象类的子类必须实现父类中的抽象方法

## 接口
- interface 关键字声明
- 接口中可以定义多个方法声明，这些方法声明不能有任何实现，并且这些方法的可见性都应该是 public，因为接口中的方法都要被其他类实现
- 实现了某个接口的类必须实现接口声明的所有方法
- 标识一个类实现某个接口通过关键字 implements 完成, 一个类可以实现多个接口
- 接口和抽象类一样，也不能被实例化，只能被其他类实现，但是和抽象类不同，接口中不包含任何具体的属性(可以配合抽象类，实现)和方法
- 可以插入一个抽象类xxx 作为中间层，来定义具体实现类的共有属性，然后让抽象类实现接口
- 接口比抽象类的抽象层级更高,在代码底层设计时，使用接口更加灵活一些。在 Laravel 框架中，大量应用了 IoC 容器和依赖注入的概念
``` php
<?php

interface CarContract
{
    public function drive();
}

abstract class BaseCar implements CarContract
{
    protected $brand;

    public function __construct($brand)
    {
        $this->brand = $brand;
    }

    // 将接口方法声明为抽象方法，让子类去实现
    abstract public function drive();
}

class LynkCo01 extends BaseCar
{
    public function __construct()
    {
        $this->brand = '领克01';
        parent::__construct($this->brand);
    }

    public function drive()
    {
        echo "启动{$this->brand}汽车" . PHP_EOL;
    }
}
```

# 修饰符
- final 还可以用于修饰类，通过 final 修饰的类将不能被子类继承。修饰方法，不能被子类重写
- static 静态方法
- abstract 抽象方法 抽象类

## 类型运算符
类型运算符 instanceof，用于判断某个对象实例是否实现了某个接口，或者是某个父类/抽象类的子类实例, 返回bool类型