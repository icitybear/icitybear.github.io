---
title: "go设计模式-装饰器模式" #标题
date: 2023-07-14T17:50:53+08:00 #创建时间
lastmod: 2023-07-14T17:50:53+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- go设计模式
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
- 通过**高阶函数**实现
- 核心思路就是在被修饰的功能模块（这里是外部传入的乘法函数 f）**<font color="red">执行前后加上一些额外的业务逻辑，而又不影响原有功能模块的执行.</font>**
- 在 main 函数中调用乘法函数 multiply 时，如果要应用装饰器，需要通过装饰器 execTime 包裹，装饰器返回的是个匿名函数，所以需要再度调用<font color="red">(延时执行)</font>才能真正执行，
# 基本功能模块
``` go
package main

import "fmt"

func multiply(a, b int) int {
    return a * b
}

func main() {
    a := 2
    b := 8
    c := multiply(a, b)
    fmt.Printf("%d x %d = %d\n", a, b, c) // 2 x 8 = 16
}
```
**不修改现有 multiply 函数代码的前提下计算乘法运算的执行时间**

# 装饰器模式
``` go
package main

import (
    "fmt"
    "time"
)

// 为函数类型设置别名提高代码可读性
type MultiPlyFunc func(int, int) int

// 乘法运算函数
func multiply(a, b int) int {
    return a * b
}

// 通过高阶函数在不侵入原有函数实现的前提下计算乘法函数执行时间
func execTime(f MultiPlyFunc) MultiPlyFunc {
    return func(a, b int) int {
        // 真正执行乘法运算函数 f 前
        start := time.Now() // 起始时间

        c := f(a, b)  // 执行乘法运算函数 用来返回函数

        end := time.Since(start) // 函数执行完毕耗时
        fmt.Printf("--- 执行耗时: %v ---\n", end)
        return c  // 返回计算结果
    }
}

func main() {
    a := 2
    b := 8
    // 通过修饰器调用乘法函数，返回的是一个匿名函数
    decorator := execTime(multiply)
    // 执行修饰器返回函数
    c := decorator(a, b)
    fmt.Printf("%d x %d = %d\n", a, b, c)
}

// 输出
// --- 执行耗时: 180ns ---
// 2 x 8 = 16
```
- 为了实现更加通用的函数执行耗时计算功能，应该将 MultiPlyFunc 函数参数和返回值声明为泛型