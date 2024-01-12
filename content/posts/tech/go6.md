---
title: "5.0-go的复合类型-指针与unsafe.Pointer" #标题
date: 2023-07-14T14:01:41+08:00 #创建时间
lastmod: 2023-07-14T14:01:41+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- go基础
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
![](pointer1.png)
# 示例
![](pointer2.png)
# 声明和初始化
- 变量的本质对一块内存空间的命名，我们可以通过引用变量名来使用这块内存空间存储的值
- 指针则是用来<font color="red">指向这些变量值所在**内存地址的值(0x0001)**</font>
- 变量值所在内存地址的值(也可能是存储指针类型的值,也叫**指针地址**)不等于该内存地址存储的变量值。<font color="red">一个变量是**指针类型**的</font>，那么就可以用这个变量来**存储指针类型的值（该类型变量的内存地址）**。
- 不能进行偏移和运算。
- 声明指针后，未初始化默认值为 nil,初始化通过&获取变量内存地址赋值
- 通过 := 对指针类型进行声明并初始化,底层会自动判断指针的类型
- <font color="red">指针变量ptr自己有也有内存地址</font>， %p 变量的内存地址 (指针本身地址)，如0x0005
- 使用场景：类型指针和切片(array unsafe.Pointer //指向存放数据的数组指针)
- 如果需要并发安全，则尽可能地不要使用指针，使用指针一定要保证并发安全,像 int、bool 这样的小数据类型没必要使用指针,指针最好不要嵌套
``` go  
a := 100
var ptr *int  // 声明指针类型 默认值为 nil
ptr = &a      // 初始化指针类型值为变量 a 
fmt.Println(ptr)
fmt.Println(*ptr)

// 输出
0xc0000a2000
100

```

代码中的 ptr 就是一个<font color="red">指针类型，表示指向**存储 int 类型值**的指针</font>。ptr 本身是一个内存地址值，所以需要通过内存地址进行赋值（通过 &a 可以获取变量 a 所在的内存地址），赋值之后，可以通过 <font color="red">*ptr (指针取值操作)获取指针指向内存地址存储的变量值（我们通常将这种引用称作「间接引用」）</font>

PHP/Java 中也有类似通过 & 进行引用传值的用法，其实这种用法的本质也是指针，只不过 PHP/Java 在语言级别屏蔽了指针的概念而已。

# 优点
- 允许对这个指针类型数据指向的内存地址存储值进行修改
- 传递数据时如果使用指针则无须拷贝数据从而节省内存空间, 提高程序的性能（指针可以直接指向某个变量值的内存地址，可以极大节省内存空间，操作效率也更高
  - 指针变量在传值时之所以可以节省内存空间，是因为指针指向的内存地址的大小是固定的,与指针指向内存地址存储的值类型(字符串 整形 结构体等)无关
  
# 通过指针传值
通过指针传值就类似于 PHP/Java 中通过引用传值
``` go  
// 先声明参数为指针类型
func swap(a, b *int)  {
    // 对应的内存空间存储值已经交换
    // *a, *b = *b, *a
    // 取a指针的值, 赋给临时变量t  指针的取值操作
    t := *a
    // 取b指针的值, 赋给a指针指向的变量
    *a = *b
    // 将a指针的值(t) 赋给b指针指向的变量
    *b = t

    fmt.Println(*a, *b)
}

func main() {
    a := 1
    b := 2
    // 本质也是拷贝了 
    swap(&a, &b)
    fmt.Println(a, b)
}

// 输出
2 1
2 1
```

# * 声明时和取值操作
- 声明时的* ,表示一个变量是T的指针类型（指针变量var ptr *int，放类型前，还有函数参数，返回值）
- 放指针变量前, * 表示一个指针变量所指向的存储单元（*ptr）,也就是指针变量指向的原变量的值（取指针地址的值）
- * 取值操作 赋值操作符的左边时，表示 a 指针指向的变量（x）= 赋值操作符的右边时，取b指针的值（取值操作，就是关联指向变量x的值）

# 视频知识点
[指针](https://www.bilibili.com/video/BV1gf4y1r79E?p=10)

# unsafe.Pointer
前面介绍的指针都是被声明为指定类型的，而 unsafe.Pointer 是特别定义的一种指针类型，它可以包含任意类型变量的地址（类似 C 语言中的 void 类型指针）

1. 任何类型的指针都可以被转化为 unsafe.Pointer；
2. unsafe.Pointer 可以被转化为任何类型的指针；
   
3. uintptr 可以被转化为 unsafe.Pointer；
4. unsafe.Pointer 可以被转化为 uintptr

## 类型转换
``` go
i := 10
var p *int = &i
// 指向 int 类型的指针转化为了 unsafe.Pointer 类型
// 再转化为 *float32 类型
var fp *float32 = (*float32)(unsafe.Pointer(p))
*fp = *fp * 10
fmt.Println(i)  // 100
```

## 指针运算实现(不安全的操作)

**为什么要单独列出这个类型uintptr?**

uintptr 是 Go 内置的可用于存储指针的整型，而整型是可以进行数学运算的,绕过 Go 指针的安全限制，实现对指针的动态偏移和计算
``` go
arr := [3]int{1, 2, 3}
ap := &arr

sp := (*int)(unsafe.Pointer(uintptr(unsafe.Pointer(ap)) + unsafe.Sizeof(arr[0])))
*sp += 3
fmt.Println(arr) // [1 5 3]

```
将数组 arr 的内存地址赋值给指针 ap，然后通过 unsafe.Pointer 这个桥梁转化为 uintptr 类型，再加上数组元素偏移量（通过 unsafe.Sizeof 函数获取），就可以得到该数组第二个元素的内存地址，最后通过 unsafe.Pointer 将其转化为 int 类型指针赋值给 sp 指针，并进行修改