---
title: "2.1-go数据类型转换" #标题
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
- 无类型常量， math.Pi 是 math 包的常量，<font color="red">默认没有类型，会在引用到的地方自动根据实际类型进行推导</font>, 范围取最大比如 uint8 < int32 < int等

# 整型之间的转化
- 有符号与无符号以及高位数字向低位数字转化时，需要注意数字的溢出和截断。
- <font color="red">浮点型转化为整型时，小数位被丢弃</font>

# 数值和布尔类型转化
自定义函数

# 将整型转化为字符串
### 整型到字符串
- 通过 Unicode 字符集转化为对应的 UTF-8 编码的字符串 <font color="red">string()</font>
- **将 byte 数组或者 rune 数组转化为字符串**,byte 是 uint8 的别名，rune 是 int32 的别名，所以也可以看做是整型数组和字符串之间的转化。
### 字符串到整形
- 不支持将字符串类型强制转化为数值类型
  
# strconv 包 封装了函数，通过包方法转换各种类型

- func Itoa(i int) string 整型转字符串
- func Atoi(s string) (i int, err error) 字符串转整型
- Parse 系列函数 将字符串转换为指定类型的值 
  - func ParseXXX(str string) (value XXX, err error)  
- Format 系列函数 将给定类型数据格式化为字符串类型的功能 
  - func FormatInt(i int64, base int) string
- Append 系列函数 将指定类型转换成字符串后追加到一个切片中

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

``` go
func main() {
    f := 901848.38

    // 将浮点数转换为字符串，保留2位小数
    str1 := strconv.FormatFloat(f, 'f', 2, 64)
    fmt.Println(str1) // 输出: 901848.38

    // 使用科学计数法表示浮点数，保留3位小数
    str2 := strconv.FormatFloat(f, 'e', 3, 64)
    fmt.Println(str2) // 输出: 9.018e+05
}
```

# protobuf3中的float

当使用Kratos框架返回int64字段时，Kratos默认会将整型字段输出为字符串。

- <font color="red">这是因为在HTTP传输中，整型数据会被序列化为JSON格式，而JSON中只支持字符串、数字、布尔值、数组和对象等数据类型。</font>所以为了保持数据的类型完整性，Kratos会将整型字段以字符串形式输出。如果需要将整型字段作为数字类型输出，可以通过类型转换等方式进行处理。这是为了避免在前端处理过程中丢失精度。如果希望将int64字段直接以数字类型输出，可以将其转换为int类型，或者在序列化过程中使用特定的库或选项来实现。例如，使用jsoniter库可以将int64字段直接序列化为数字类型

- 发起请求的参数数字，json序列化成json数字
encoding/json解码器将JSON数据解码为Go语言中的数据结构。当解码器遇到一个JSON数字时，它会将其解码为Go语言中的float64类型。
这是因为在<font color="red">JSON中，数字是以字符串的形式表示的。而在Go语言中，数字类型为float64，因为它可以容纳大部分的数字范围。</font>因此，解码器将JSON中的数字解码为float64类型以保持数据的精确性和完整性。



## go float32导致901848.38 变为901848.4

- gorm的相关modle模型 定义float64, 能正常接收，<font color="red">但是proto3 只有float类型（对应是float32），
高精度转低精度(精度丢失问题)</font>, 四舍五入问题 float32强转的精度问题

``` go
message MediaDailyCost {
	int32 id = 1;
	string dt = 2;
  // ...
	string mediaCost = 13; // float mediaCost = 13;
	string updateTime = 14;
}

// autoUpdateTime
type MediaDailyCost struct {
	ID                  uint64    `json:"id" gorm:"column:id"`
	Dt                  string    `json:"dt" gorm:"column:dt"`                                         // 日期YYYY-MM-DD
	ChannelType         int8      `json:"channel_type" gorm:"column:channel_type"`                     // 渠道类型 信息流 商城 搜索广告 cpa
	MediaCode           string    `json:"media_code" gorm:"column:media_code"`                         // 媒体代码
  // ...
	MediaCost           float64   `json:"media_cost" gorm:"column:media_cost"`                         // 媒体日消耗
	CreateTime          time.Time `json:"create_time" gorm:"autoCreateTime"`                           // 创建时间
	UpdateTime          time.Time `json:"update_time" gorm:"autoUpdateTime"`                           // 更新时间
}

mediaCostStr := strconv.FormatFloat(float64(item.MediaCost), 'f', 2, 64)
log.Context(ctx).Debugf("db:%+v, to32:%+v, to64:%+v, str:%s", item.MediaCost, float32(item.MediaCost), float64(item.MediaCost), mediaCostStr)
// "message":"db:901848.38, to32:901848.4, to64:901848.38 str:901848.38
```

- <font color="red">所以建议proto3的float类型，定义返回时可以用字符串string (int64和float的坑)</font>