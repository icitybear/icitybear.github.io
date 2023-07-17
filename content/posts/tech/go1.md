---
title: "go变量与常量，基础类型" #标题
date: 2023-07-11T13:44:02+08:00 #创建时间
lastmod: 2023-07-11T13:44:02+08:00 #更新时间
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
# 变量
- 变量命名规则遵循驼峰命名法，即首个单词小写，每个新单词的首字母大写，如 userName，但如果你的全局变量希望能够被外部包所使用，则需要将首个单词的首字母也大写。
- 关键字 var
- 只声明，不赋值，默认是0 变量在声明之后，系统会自动将变量值<font color="red">初始化为对应类型的零值</font>
- int 为 0，float 为 0.0，bool 为 false，string 为空字符串，切片、函数、指针变量的默认为 nil 等。所有的内存在 Go 中都是经过初始化的
- 先声明 再赋值
- 声明的同时赋值
- := 没声明类型 自动推断
  - 自动推断 会默认更大范围的类型， uint8会推断成int
  - 短变量声明赋值 <font color="red">只能用在函数内部的写法</font> :=
  - <font color="red">多重赋值变量</font>的左值和右值按从左到右的顺序赋值, i, j = j, i , 配合<font color="red">匿名变量_</font>
  - 推导声明写法的左值变量必须是没有定义过的变量。若定义过，将会发生编译错误
- 批量声明 var ()
- 声明赋值时，注意整型范围 超过的话会提示错误
- 作用域：局部变量与全局变量的优先级 全局跨包变量首字母要大写, 未声明局部的时候直接使用全局
- 不同类型的值不能使用 == 或 != 运算符进行比较,比较的时候类型也要相等，int32或者rune != int      mismatched types rune and int
- <font color="red">Go 是强类型语言，变量类型一旦确定，就不能将其他类型的值赋值给该变量。只支持显示转化，强转的基础也是底层要一样，比如都是整形byte<=>uint8 (字符串也就是数组) int32=>int等</font>
  ` （只能显示转封装函数，返回新值。。go不支持隐式转）比如 func itob(i int) bool { return i != 0 }   int转bool` 
- 变量逃逸概念
- 生命周期 函数的参数变量和返回值都是局部变量，它们在函数每次被调用的时候创建。
  

# 常量
- 与变量大差不差
- 常量是指编译期间就已知且不可改变的值，常量只可以是数值类型（包括整型、 浮点型和复数类型）、布尔类型、字符串类型等<font color="red">标量类型</font>
- 关键字 const, 批量
- 无类型常量, 自动推断
- 作用域，全局跨包常量首字母要大写
``` go
const Pi float64 = 3.14159265358979323846 
const zero = 0.0 // 无类型浮点常量 
const (          // 通过一个 const 关键字定义多个常量，和 var 类似
    size int64 = 1024
    eof = -1  // 无类型整型常量 
) 
const u, v float32 = 0, 3  // u = 0.0, v = 3.0，常量的多重赋值 
const a, b, c = 3, 4, "foo" // a = 3, b = 4, c = "foo", 无类型整型和字符串常量
``` 
## 预定义常量
- Go 语言预定义了这些常量：true、false 和 iota
- iota 比较特殊，可以被认为是一个可被编译器修改的常量，在每一个 const 关键字出现时被重置为 0，然后在下一个 const 出现之前，每出现一次 iota，其所代表的数字会自动增 1。如果两个 const 的赋值语句的表达式是一样的，那么还可以省略后一个赋值表达式(相同的表达式)
- 通过const实现枚举
``` go
const (
	Readable = 1 << iota //左移位
	Writable
	Executable
)
```


# 布尔型
- 类型关键字为 bool，可赋值且只可以赋值为预定义常量 true 和 false
- ! 运算符也不能作用于非布尔类型值
- 布尔类型不能接受其他类型的赋值，也不支持自动或强制的类型转换。
- 无法参与数值运算
-  &&（AND）和 ||（OR）操作符结合，并且有短路行为

# 整型
- 变量运算, 类型不一致 会提示int与uint8不匹配 不能运算，所以加强转 uint8(int x) / unit8
- 如果是大转小 会存在精度丢失（截断）的情况。强转时如果范围大于类型值会截取低8bit都是0 256就是0  比如 c = 256 uint8(c)值为0
- 位数相关, int uint uintptr 跟平台相关 字节数 1byte = 8 bit
- 通过增加前缀 0 来表示八进制数（如：077），增加前缀 0x 来表示十六进制数（如：0xFF），以及使用 E 来表示 10 的连乘（如：1E3 = 1000）。
  
## 运算
- 在 Go 语言中，也支持自增/自减运算符，即 ++/--，但是只能作为语句，不能作为表达式，<font color="red">且只能用作后缀，不能放到变量前面</font>, 支持快捷写法
- 位运算 &^ 按位置零 & | ^(异或与取反) << >>
- 逻辑运算 && || ! (只放在bool)
- 比较运算符会考虑变量的类型，<font color="red">各种类型的整型变量都可以直接与字面常量进行比较</font>, >、<、==、>=、<= 和 != 运算结果是布尔值

  
# 字符与字符串
- 双引号表示一个字符串，双引号内字符可以转义
- 单引号表示单字符 双引号标识字符串，单引号只能用来包裹<font color="red">一个字节的ASCII码字符byte</font>,也可以是多字节的字符 rune， 因为中文是多字节的所以必须用rune 4字节
- byte uint8的别名 1个字节的字符
- rune int32的别名 4个字节的字符  为啥不是uint32?是因为int32的范围够表示4个字节的字符了
- len函数 获取<font color="red">字节数</font>,\t和空格各算一个字节 中文3个字节


``` go
func TestStringByteRune(t *testing.T) {
	s0 := "中国\ta bc"
	fmt.Printf("值=%v, 类型是%T\n", s0, s0)
	s1 := []rune(s0) //字符串 中 转成 rune unicode码点
	fmt.Printf("值=%v, 类型是%T\n", s1, s1)
	s2 := []byte(s0) //字符串 中 转成 byte字节切片
	fmt.Printf("值=%v, 类型是%T\n", s2, s2)
	// 遍历切片
	for _, s := range s2 {
		fmt.Printf("uint8: %c  %d\n", s, s)
	}
}
``` 

# 字符串
- 初始化为默认零值“”
- <font color="red">字符串是byte字节的定长数组，且有序。</font>可以通过[]type(), []rune() 转成切片, 切片用range遍历, 数组与切片都可以用下标访问 ([]rune()转成 rune unicode码点  utf8编码, 访问下表正常打印)
- 不可更改 cannot assign to str[0]
- 在方括号[]内写入索引，索引从 0 开始计数，默认不强转成切片的情况下 （只对纯 ASCII 码 （[]byte()）的字符串有效）
- 字符串可以包含任意的二进制数据
- unsafe.Sizeof返回变量在内存中占用的字节数(切记，如果是slice，则不会返回这个slice在内存中的实际占用长度)
- 双引号""来定义字符串  字符串字面量（string literal）,使用`反引号， 在`间的所有代码均不会被编译器识别
- 获取字符串中某个字节的地址属于非法行为，例如 &str[i]
- utf8.RuneCountInString()个数 字节数len

## 拼接
1. 方法1：通过‘+’号，两个字符串 s1 和 s2 可以通过 s := s1 + s2 拼接在一起。将 s2 追加到 s1 尾部并生成一个新的字符串 s。（缺点不高效，字符串是不可变类型（值类型），内存拷贝 多次调用（旧的内存依旧存在）对GC产生压力）
2. 方法2：bytes.Buffer 是可以缓冲并可以往里面写入各种字节数组的。字符串也是一种字节数组，使用 WriteString() 方法进行写入（写入的时候会自动扩容）。写入 stringBuilder 中，然后再通过 stringBuilder.String() 方法将缓冲转换为字符串。1.10版本新加的包 不会增加内存开销 与旧版本的bytes.Buffer的API一样 
3. 方法3：strconv.Itoa() 每次都会新建一个字符串
- 子字符串 通过字符串切片实现获取子串
- strings标准库，字符串比较、是否包含指定字符/子串、获取指定子串索引位置、字符串替换、大小写转换、trim 等操作
4. fmt.sprintf() 本质

## 遍历
  - 一种是以字节数组的方式遍历,依据下标取字符串中的字符,类型为type,是以 UTF-8 编码的角度切入的, 此时是字节存储的utf8编码的值
  - 一种是以 Unicode 字符遍历,因为以 Unicode 字符方式遍历时，每个字符的类型是 rune，而不是 byte。通过 range 关键字遍历字符串时，又是从 Unicode 字符集的角度切入, Unicode 字符值就是数字id

``` go
str := "Hello, 世界" 
n := len(str) 
for i := 0; i < n; i++ {
    ch := str[i]    // 依据下标取字符串中的字符，ch 类型为 byte
    fmt.Println(i, ch) 
}

0 72 
1 101 
2 108 
3 108 
4 111 
5 44 
6 32 
7 228 // 注意这里 后续是 世界 utf8编码存的每个字节值
8 184 
9 150 
10 231 
11 149 
12 140

str := "Hello, 世界" 
for i, ch := range str { 
    fmt.Println(i, ch)    // ch 的类型为 rune 
    // fmt.Println(i, string(ch)) // 将 Unicode 字符编码转化为对应的字符
}

0 72 
1 101 
2 108 
3 108 
4 111 
5 44 
6 32 
7 19990 // 注意这里 后续是 世界 是Unicode 中的字符的 ID 可以通过string函数转化
10 30028

// 将 Unicode 字符编码转化为对应的字符
0 H
1 e
2 l
3 l
4 o
5 ,
6  
7 世
10 界

```

# 字符串底层
## 字符集与编码规则
- 字符集为每个字符分配一个唯一的 ID，我们使用到的所有字符在 Unicode 字符集中都有一个唯一的 ID，例如上面例子中的 a 在 Unicode 与 ASCII 中的编码都是 97。汉字“你”在 Unicode 中的编码为 20320，在不同国家的字符集中，字符所对应的 ID 也会不同。<font color="red">而无论任何情况下，Unicode 中的字符的 ID 都是不会变化的。</font>
**UTF-8 是编码规则，将 Unicode 中字符的 ID 以某种方式进行编码**

## Go 语言中支持两种<font color="red">字符</font>类型 byte和rune
- 从 Unicode 字符集的视角看，字符串的每个字符都是一个字符的独立单元，但如果从 UTF-8 编码的视角看，一个字符可能是由多个字节编码而来的。
- 在 Go 语言中可以通过 unicode/utf8 包进行 UTF-8 和 Unicode 之间的转换。
- Go语言中字符串的实现基于 UTF-8 编码(编码规则)，通过 rune 类型（int32），可以方便地对每个 UTF-8 字符进行访问。当字符为 ASCII 码时则占用 1 个字节，其它字符根据需要占用 2-4 个字节，比如中文编码通常需要 3 个字节。如果单纯只有ASCII码那转成byte
- Go 代码需要包含非 ANSI 字符，保存源文件时请注意编码格式必须选择 UTF-8
- Go 语言默认仅支持 UTF-8 和 Unicode 编码，对于其他编码，Go 语言标准库并没有内置的编码转换支持(开源封装)[iconv 库](https://github.com/qiniu/iconv)
- 出于简化语言的考虑，Go 语言的多数 API 都假设字符串为 UTF-8 编码。
- 将 Unicode 字符编码转化为对应的字符，<font color="red">可以使用 string 函数进行转化</font>, 但是如果是UTF-8 编码不能这样转化，英文字符没问题，因为一个英文字符就是一个字节，中文字符则会乱码，因为一个中文字符编码需要三个字节，转化单个字节会出现乱码。