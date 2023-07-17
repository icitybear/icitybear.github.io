---
title: "go数据类型转换" #标题
date: 2023-07-12T22:25:36+08:00 #创建时间
lastmod: 2023-07-12T22:25:36+08:00 #更新时间
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

# 数据类型转换
- 由于Go语言不存在隐式类型转换，因此所有的类型转换都必须显式的声明
- <font color="red">类型转换精度问题</font>，从一个取值范围较小的类型转换到一个取值范围较大的类型（将 int16 转换为 int32）。当从一个取值范围较大的类型转换到取值范围较小的类型时（将 int32 转换为 int16 或将 float32 转换为 int），会发生精度丢失（截断）的情况。
- 只有<font color="red">相同底层类型的变量之间可以进行相互转换</font>（如将 int16 类型转换成 int32 类型），不同底层类型的变量相互转换时会引发编译错误（如将 bool 类型转换为 int 类型），这种的要<font color="red">通过函数通过输入输出转换（自定义函数强转）</font>
  - 布尔类型不能接受其他类型的赋值，也不支持自动或强制的类型转换。
- 无类型常量， math.Pi 是 math 包的常量，<font color="red">默认没有类型，会在引用到的地方自动根据实际类型进行推导</font>

# 整型之间的转化
- 有符号与无符号以及高位数字向低位数字转化时，需要注意数字的溢出和截断。
- 浮点型转化为整型时，小数位被丢弃

# 数值和布尔类型转化
自定义函数

# 将整型转化为字符串
### 整型到字符串
- 通过 Unicode 字符集转化为对应的 UTF-8 编码的字符串 string()
- 将 byte 数组或者 rune 数组转化为字符串,byte 是 uint8 的别名，rune 是 int32 的别名，所以也可以看做是整型数组和字符串之间的转化。
### 字符串到整形
- 不支持将字符串类型强制转化为数值类型
  
# strconv 包 封装了函数，通过包方法转换各种类型
``` go
v1 := "100"
v2, _ := strconv.Atoi(v1)  // 将字符串转化为整型，v2 = 100

v3 := 100
v4 := strconv.Itoa(v3)   // 将整型转化为字符串, v4 = "100"

v5 := "true"
v6, _ := strconv.ParseBool(v5)  // 将字符串转化为布尔型
v5 = strconv.FormatBool(v6)  // 将布尔值转化为字符串

v7 := "100"
v8, _ := strconv.ParseInt(v7, 10, 64)   // 将字符串转化为整型，第二个参数表示进制，第三个参数表示最大位数
v7 = strconv.FormatInt(v8, 10)   // 将整型转化为字符串，第二个参数表示进制

v9, _ := strconv.ParseUint(v7, 10, 64)   // 将字符串转化为无符号整型，参数含义同 ParseInt
v7 = strconv.FormatUint(v9, 10)  // 将无符号整数型转化为字符串，参数含义同 FormatInt

v10 := "99.99"
v11, _ := strconv.ParseFloat(v10, 64)   // 将字符串转化为浮点型，第二个参数表示精度
v10 = strconv.FormatFloat(v11, 'E', -1, 64)

q := strconv.Quote("Hello, 世界")    // 为字符串加引号
q = strconv.QuoteToASCII("Hello, 世界")  // 将字符串转化为 ASCII 编码
```