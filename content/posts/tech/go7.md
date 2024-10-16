---
title: "7.0-go的流程控制" #标题
date: 2023-07-14T16:03:59+08:00 #创建时间
lastmod: 2023-07-14T16:03:59+08:00 #更新时间
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

# 分支结构 if else
- if 还有一种特殊的写法，可以在 if 表达式之前添加一个执行语句，再根据变量值进行判断
  - 这种写法可以将返回值与判断放在一行进行处理，而且返回值的作用范围被限制在 if、else 语句组合中。
- 没有elseif
``` go
if condition {
    // do something
}

if condition {
    // do something
} else {  
    // do something
}

if condition1 {
    // do something
} else if condition2 {  //没有elseif
    // do something else
} else { 
    //关键字 if 和 else 之后的左大括号{必须和关键字在同一行，如果你使用了 else if 结构，则前段代码块的右大括号}
    必须和 else if 关键字在同一行 gofmt 格式化会自动格式化
    // catch-all or default
}

// if 还有一种特殊的写法，可以在 if 表达式之前添加一个执行语句，再根据变量值进行判断
if err := Connect(); err != nil {
    fmt.Println(err)
    return
}
```

# 循环结构 for
1. 只支持 for 关键字，<font color="red">而不支持 while 和 do-while 结构</font>
2. 每次循环开始前都会计算条件表达式，如果表达式为 true，则循环继续，否则结束循环，条件表达式可以被忽略，忽略条件表达式后默认形成<font color="red">无限循环。</font>
3. for 的<font color="red">结束语句</font>为 i++，每次结束循环前都会调用,如果循环被 break、goto、return、panic 等语句强制退出，结束语句不会被执行。
4. 只有一个循环条件的循环, 忽略条件的死循环
5. 允许在循环条件中定义和初始化变量，如果变量在此处被声明，其<font color="red">作用域将被局限在这个 for 的范围内</font>
6. 唯一的区别Go语言不支持以逗号为间隔的多个赋值语句，必须使用平行赋值<font color="red">(多重赋值)的方式来初始化多个变量</font>
``` go
var i int
//结束循环时带可执行语句的无限循环
for ; ; i++ {
    if i > 10 {
        break
    }
}
//无限循环
var i int
for {
    if i > 10 {
        break
    }
    i++
}
//只有一个循环条件的循环
var i int
for i <= 10 {
    i++
}

//允许在循环条件中定义和初始化变量，如果变量在此处被声明，其作用域将被局限在这个 for 的范围内
//唯一的区别Go语言不支持以逗号为间隔的多个赋值语句，必须使用平行赋值(多重赋值)的方式来初始化多个变量
for j := 0; j < 5; j++ {
    for i := 0; i < 10; i++ {
        if i > 5 {
            //break 语句终止的是 JLoop 标签处的外层循环。
            break JLoop
        }
        fmt.Println(i)
    }
}
JLoop:
// ...

```

# 键值循环 for range

for range 结构是Go语言特有的一种的迭代结构，在许多情况下都非常有用，for range 可以遍历字符串、数组、切片、map （遍历时无序）及通道（channel, 只有一个val值）, range 语法上类似于其它语言中的 foreach 语句
1. 数组、切片、字符串返回索引和值。 数值切片也是有索引的0开始
2. map 返回键和值。对 map 遍历时，<font color="red">遍历输出的键值是无序的</font>，如果需要有序的键值对输出，需要对结果组合成切片再进行排序。
3. 通道（channel）只返回通道内的值。<font color="red">通道在遍历时，只输出一个值</font>，即管道内的类型对应的数据。  v:= range ch
4. <font color="red">val 始终为集合中对应索引的值拷贝</font>，因此它一般只具有只读性质，对它所做的任何修改都不会影响到集合中原有的值。
5. 匿名变量本身不会进行空间分配，也不会占用一个变量的名字。

## range 切片+指针 v只初始化一次
- v 只初始化了一次，之后的遍历都是在原来遍历的基础上赋值，所有v的指针（地址）并没有变。该指针指向的是最后一次遍历的v的值
``` go
type Person struct {
	name string
}

func main() {
	arr := []Person{
		Person{"小明"},
		Person{"小刚"},
	}
	var res []*Person

	for _, v := range arr {
        fmt.Printf("v 指针 %p\n", &v)
		fmt.Println("v 的值", v)
		res = append(res, &v)
	}
    // 正确写法
    // for i, _ := range arr {
	// 	fmt.Printf("v 指针 %p\n", &arr[i])
	// 	fmt.Println("v 的值", arr[i])
	// 	res = append(res, &arr[i])
	// }
    fmt.Println(res)
	// 遍历查看结果集
	for _, person := range res{
		fmt.Println("name-->:", person.name)
	}
}

// v 指针 0xc0001101e0
// v 的值 {小明}
// v 指针 0xc0001101e0
// v 的值 {小刚}
// [0xc0001101e0 0xc0001101e0]
// name-->: 小刚
// name-->: 小刚
```

## 新版1.22

1.不再共享循环变量
``` go
func main() {
	values := []int{1, 2, 3, 4, 5}
	for _, value := range values {
		go func() {
			fmt.Printf("%p,%d\n", &value, value)
		}()
	}
	time.Sleep(time.Second * 3)
}

0xc00000a0d8,4
0xc00000a0d8,5
0xc00000a0d8,5
0xc00000a0d8,3
0xc00000a0d8,5
// 1.22之后
0xc00000a0f0,2
0xc00000a108,5
0xc00000a100,4
0xc00000a0d8,1
0xc00000a0f8,3
```
2. 支持循环整形类型
``` go
func main() {
    //1.22前 cannot range over 5 (untyped int constant)
	for i := range 5 {
		fmt.Println("Hello World!", i) 
	}
}

Hello World! 0
Hello World! 1
Hello World! 2
Hello World! 3
Hello World! 4
```

# switch case语句

- 表达式不需要为常量，甚至不需要为整数,<font color="red">为表达式</font> case r > 10 && r < 20  字符串 整型匹配等
- case 与 case 之间是独立的代码块，<font color="red">不需要通过 break</font> 语句跳出当前 case 代码块以避免执行到下一行
- Go语言规定每个 switch 只能有一个 default 分支
- switch的<font color="red">每一个case是从上到下去匹配</font>，如果没有break，则每一个都会去匹配
- 不同的 case 表达式使用<font color="red">逗号分隔</font>。 case "mum", "daddy":
- Go里面switch默认相当于每个case最后带有break，匹配成功后不会自动向下执行其他case，而是跳出整个switch, 但是可以使用fallthrough强制执行后面的case代码。
  - fallthrough不能用在switch的最后一个分支
  - 加了fallthrough后，会直接运行【紧跟的后一个】case或default语句，不论条件是否满足都会执行，后面的条件并不会再判断了
  
``` go
var a = "hello"
switch a {
case "hello":
    fmt.Println(1)
case "world":
    fmt.Println(2)
default:
    fmt.Println(0)
}
```

``` go
s := "abcd" 
switch s[1] {
case 'a' 
    fmt.Println("the integer was <=4") 
    fallthrough
case 'b': 
    fmt.Println("The integer was <= 5") 
    fallthrough // 会直接运行【紧跟的后一个】case或default语句，不论条件是否满足都会执行
case 'c': 
    fmt.Println("The integer was <= 6") 
default:
    fmt.Println("default case")
}

// 输出
// The integer was <= 5
// The integer was <= 6
```

# 踩坑点
## case 多个条件时
``` go
// 错误写法，php惯性
func TestSwitchCaseCondition2(t *testing.T) {
	i := 3
	switch i {
	case 3: // 并不会执行到下面的case
	case 4:
		t.Log(i) // 只有4会执行，3不会
	default:
		t.Log("unknow")
	}
	t.Log("after switch")
}
// 正确写法
func TestSwitchCaseCondition2(t *testing.T) {
	i := 3
	switch i {
	case 3, 4:
		t.Log(i) // 3和4都会执行
	default:
		t.Log("unknow")
	}
	t.Log("after switch")
}
```

## 退出多层循环 break, goto, continue
- Go语言也支持label(标签)语法：分别是break label和 goto label 、continue label
- 一般通过break 多次，或者通过break 标签名 ，<font color="red">标签要求必须定义在对应的 for、switch 和 select 的代码块上</font>，其他语言break n，

- goto 语句通过标签进行代码间的无条件跳转  退出多重循环，这个功能会影响代码的可读性， 会让代码结构看起来比较乱。
  - <font color="red">只建议在标签在最下面使用， 上面的时候进行跳转到标签 goto</font>
  - <font color="red">后续如果有使用局部变量，那么该局部变量定义旧的就得放在最前面，很麻烦，</font>

``` go
// 报错 goto xxx jumps over variable declaration 
func main() {
    goto Label
    x := 5
Label:
    fmt.Println(x)
}
// 正确的
func main() {
    var x int // 部变量定义旧的就得放在最前面
    goto Label
    x = 5
Label:
    fmt.Println(x) // 请注意，这里 x 没有被初始化，因此它的值为 int 的零值，即 0
}
```

``` go
func main() {

OuterLoop:
    for i := 0; i < 2; i++ {
        for j := 0; j < 5; j++ {
            // 偶而也配合select
            switch j {
            case 2:
                fmt.Println(i, j)
                break OuterLoop
            case 3:
                fmt.Println(i, j)
                break OuterLoop
            }
        }
    }
}
```


- <font color="red">for配合select或者switch, break只是跳出该次循环 跳出当前select, 继续下次循环</font>

``` go
func TestForSelect(t *testing.T) {
	// for配合select break只是跳出select, 继续下次循环， 相当continue
	for i := 0; i < 5; i++ {
		switch {
		case i%2 == 0:
			t.Log("Even")
		case i%2 == 1:
			t.Log("Odd")
			break // 继续下次循环， 相当continue 没有break n
			t.Log("hhh") // 后续不执行
		default:
			t.Log("unknow")
		}
	}
}
```

# continue（继续下一次循环）
continue 语句可以结束当前循环，开始下一次的循环迭代过程，<font color="red">**仅限在 for 循环内使用**</font>，在 continue 语句后添加标签时，表示开始标签对应的循环。for循环里的switch可以使用break