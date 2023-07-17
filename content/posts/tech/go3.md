---
title: "go的类型概述" #标题
date: 2023-07-13T22:58:38+08:00 #创建时间
lastmod: 2023-07-13T22:58:38+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- go基础
- go面向对象
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

# 简洁性

- Go 语言并没有沿袭传统面向对象编程中的诸多概念，比如类的继承、接口的实现、构造函数和析构函数、隐藏的 this 指针等，也没有 public、protected、private 之类的访问修饰符。
- 整个类型系统通过接口串联

# 类型系统
- 基本类型，如 byte、int、bool、float、string 等；
- 复合类型，如数组、切片、字典、指针、结构体等；
- 可以指向任意对象的类型（Any 类型），其实就是空接口
- 值语义和引用语义；值语义和引用语义等价于之前介绍类型时提到的值类型和引用类型。
- 面向对象，即所有具备面向对象特征（比如成员方法）的类型
- 接口。关键字interface

## 值类型
  - 基本类型，如布尔类型、整型、浮点型、字符串等；
  - 复合类型，如数组、结构体等
  1. 所有值语义类型都支持定义成员方法，包括内置基本类型。
  2. 基本类型特殊，需要将基本类型通过 type 关键字设置为新的类型
``` go
type Integer int

func (a Integer) Equal(b Integer) bool {
    return a == b
}
// 加法
func (a Integer) Add(b Integer) Integer {
    return a + b
}

// 乘法
func (a Integer) Multiply(b Integer) Integer {
    return a * b
}

func main() {
    var x Integer
    var y Integer
    x, y = 10, 15
    fmt.Println(x.Equal(y))
}
```
**<font color="red">本质上它仍然是一个整型数字，你可以把具体的数值看做是 Integer 类型的实例</font>,将基本的数字类型就转化成了面向对象类型。**

## 引用类型
  - 切片、字典、指针和通道都是引用语义

## 接口
**实现某个接口时，只需要实现该接口要求的所有方法即可，<font color="red">无需显式声明实现的接口</font>**
``` go
type Math interface {
    Add(i Integer) Integer
    Multiply(i Integer) Integer
}
```
结合上面 Integer类型，认为 Integer 实现了 Math 接口，无需显式声明。

## 任意类型
- 任何类型都可以被 Any 类型引用
- Any 类型就是空接口，即 interface{}