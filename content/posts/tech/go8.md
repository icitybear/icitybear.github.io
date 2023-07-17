---
title: "go函数" #标题
date: 2023-07-14T16:34:24+08:00 #创建时间
lastmod: 2023-07-14T16:34:24+08:00 #更新时间
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

- <font color="red">一等公民</font>,函数本身可以作为值进行传递,支持匿名函数和闭包（closure）,函数可以满足接口
- <font color="red">三种类型</font>的函数普通的带有名字的函数 ,匿名函数或者lambda 函数, 类方法
- 函数不是变量，存在内存地址,<font color="red"> 函数名的本质就是一个指向其函数内存地址的指针常量</font>
# 声明
- 函数的基本组成为：关键字 func、函数名、参数列表、返回值、函数体和返回语句
- 编译型语言，所以函数编写的顺序是无关紧要的
- 函数变量——把函数作为值保存到变量中 变量是函数类型
  
## 返回值
- return 语句可以带有零个或多个参数，这些参数将作为返回值供调用者使用，简单的 return 语句也可以用来结束 for 的死循环，或者结束一个协程（goroutine）
- 使用多返回值中的最后一个返回参数,一般返回函数执行中可能发生的错误
- 可以无返回值 返回值列表的括号是可以省略的
- 多个返回值类型括起来，用逗号分隔每个返回值的类型，使用 return 语句返回时，值列表的顺序需要与函数声明的返回值类型一致
- 同一种类型返回值  (int, int)
- 命名返回值 (a, b int)命名的返回值变量的默认值为类型的默认值，即数值为 0，字符串为空字符串，布尔为 false、指针为 nil 等

## 可变参数 变长参数
变长参数指的是函数参数的数量不确定，可以按照需要传递任意数量的参数到指定函数
- <font color="red">...type 语法糖</font>
- 形如 ...type 格式的类型只能作为函数的参数类型存在，**并且必须是函数的最后一个参数**
``` go
// 可变参数类型约束为 int，如果你希望传任意类型，可以指定类型为 interface{}
// 在参数类型前加上 ... 前缀，就可以将该参数声明为变长参
func myfunc(ops ...int) int {
    ret := 0
    //从内部实现机理上来说，类型...type本质上是一个数组切片，也就是[]type，
    //这也是为什么上面的参数 args 可以用 for 循环来获得每个传入的参数。
    for _, op := range ops {
        ret += op
    }
    return ret
}

//slist ...interface{}
myfunc(1, 2, 3, 4, 5) 

slice := []int{1, 2, 3, 4, 5}
myfunc(slice...)
myfunc(slice[1:3]...)
```

# 调用函数
- 每一次函数在调用时都必须按照声明顺序为所有参数提供实参（参数值）
- 在函数调用时，Go语言<font color="red">没有默认参数值</font>
## 值传承
函数接收到传递进来的参数后，会将参数值拷贝给声明该参数的变量（也叫形式参数，简称形参）
## 引用传参 形参 类型要申明 指针类型 *T
传递给函数的参数是一个指针，而指针代表的是实参的内存地址，修改指针引用的值即修改变量内存地址中存储的值，所以实参的值也会被修改（这种情况下，<font color="red">传递的是变量地址值的拷贝</font>，所以从本质上来说还是按值传参）：

# 常见内置函数
提供了很多不需要导入任何包就可以直接调用的内置函数: len cap make new append copy panic recover close

# 匿名函数
没有指定函数名的函数声明方式
``` go
// 1、将匿名函数赋值给变量
add := func(a, b int) int {
    return a + b
}
// 调用匿名函数 add
fmt.Println(add(1, 2))  
// 2、定义时直接调用匿名函数
func(a, b int) {
    fmt.Println(a + b)
} (1, 2) 
```

## 闭包
- 闭包指的是引用了自由变量（未绑定到特定对象的变量，通常在函数外定义）的函数
- 即使外部状态已经失效，闭包内部依然保留了一份从外部引用的变量。
- <font color="red">有状态的匿名函数, 匿名函数引用了外部变量，就形成了一个闭包（Closure）</font>
- Go 函数, 可以赋值给变量，也可以作为参数传递给其他函数，还能够被函数动态创建和返回。

第一类对象指的是运行期可以被创建并作为参数传递给其他函数或赋值给变量的实体，在绝大多数语言中，数值和基本类型都是第一类对象，在支持闭包的编程语言中（比如 Go、PHP、JavaScript、Python 等），函数也是第一类对象

``` go
func main() {
    ...

    // 普通的加法操作
    add1 := func(a, b int) int {
        return a + b
    }

    // 定义多种加法算法
    base := 10
    add2 := func(a, b int) int {
        return a * base + b // 引用了自由变量base 形成了一个闭包
    }

    handleAdd(1, 2, add1) // 3
    handleAdd(1, 2, add2) // 12 
    // handleAdd 外部函数时传入了闭包 add2 作为参数，add2 闭包在外部函数中执行时，虽然作用域离开了 main 函数，但是还是可以访问到变量 base
}

// 将匿名函数作为参数
func handleAdd(a, b int, call func(int, int) int) {
    fmt.Println(call(a, b))
}
```

## 用处场景
- 匿名函数内部声明的局部变量无法从外部修改
- (高阶函数)将匿名函数<font color="red">作为函数参数</font>, Go 官方 net/http 包底层的路由处理器也是这么实现的
- (高阶函数)将匿名函数<font color="red">作为函数返回值</font>
``` go
// 将函数作为返回值类型
func deferAdd(a, b int) func() int {
    return func() int {
        return a + b // 匿名函数引用了外部函数传入的参数a b，因此形成闭包
    }
}

func main() {
    ...

    // 此时返回的是匿名函数
    addFunc := deferAdd(1, 2)
    // 这里才会真正执行加法操作
    fmt.Println(addFunc())
}
```

**<font color="red">通过将函数返回值声明为函数类型来实现业务逻辑的延迟执行</font>, 这里只有真正运行 返回的匿名函数，才真正执行加法操作,让执行时机完全掌握在开发者手中**

# 以下函数式编程技术

# 高阶函数
高阶函数，就是接收其他函数作为参数传入，或者把其他函数作为结果返回的函数
## 实现装饰器模式
{{< innerlink src="posts/tech/go9.md" >}}  

# 递归函数
1. 一个问题的解可以被拆分成多个子问题的解
2. 拆分前的原问题与拆分后的子问题除了数据规模不同，求解思路完全一样
3. 子问题存在递归终止条件

[优化例子](https://geekr.dev/posts/go-recursive-function-and-optimization)

# Map-Reduce
**处理<font color="red">数组、切片、字典等集合类型</font>，常规做法都是循环迭代进行处理**

缺点：针对简单的单个场景，这么实现没什么问题，但这是典型的面向过程思维，而且代码几乎没有什么复用性可言

**<font color="red">Map-Reduce 模型：先将字典类型切片转化为一个字符串类型切片（Map，字面意思就是一一映射），再将转化后的切片元素转化为整型后累加起来（Reduce，字面意思就是将多个集合元素通过迭代处理减少为一个）。</font>**
- 一般处理
``` go
package main

import (
    "fmt"
    "strconv"
)

func ageSum(users []map[string]string) int {
    var sum int
    for _, user := range users {
        num, _ := strconv.Atoi(user["age"])
        sum += num
    }
    return sum
}

func main() {
    var users = []map[string]string{
        {
            "name": "张三",
            "age": "18",
        },
        {
            "name": "李四",
            "age": "22",
        },
        {
            "name": "王五",
            "age": "20",
        },
    }
    fmt.Printf("用户年龄累加结果: %d\n", ageSum(users))
}
```
- 引入Map-Reduce 

``` go
// 选择性过滤器
func itemsFilter(items []map[string]string, f func(map[string]string) bool) []map[string]string {
    newSlice := make([]map[string]string, len(items))
    for _, item := range items {
        if f(item) {
            newSlice = append(newSlice, item)
        }
    }
    return newSlice
}

// map转切片
func mapToString(items []map[string]string, f func(map[string]string) string) []string {
    newSlice := make([]string, len(items))
    for _, item := range items {
        newSlice = append(newSlice, f(item))
    }
    return newSlice
}

// 也是迭代处理，但是处理方式 为函数参数
func fieldSum(items []string, f func(string) int) int {
    var sum int
    for _, item := range items{
        sum += f(item)
    }
    return sum
}
func main() {
    var users = []map[string]string{
        {
            "name": "张三",
            "age": "18",
        },
        {
            "name": "李四",
            "age": "22",
        },
        {
            "name": "王五",
            "age": "20",
        },
    }
    // 可追加的过滤器, 返回的还是map
    validUsers := itemsFilter(users, func(user map[string]string) bool {
        age, ok := user["age"]
        if !ok {
            return false
        }
        intAge, err := strconv.Atoi(age)
        if err != nil {
             return false
        }
        if intAge < 18 || intAge > 35 {
            return false
        }
        return true
    })

    // 返回切片 如果不过滤 第一个参数 users
    ageSlice := mapToString(validUsers, func(user map[string]string) string {
        return user["age"]
    })
    // 对切片 进行处理
    sum := fieldSum(ageSlice, func(age string) int {
        intAge, _ := strconv.Atoi(age)
        return intAge
    })
    fmt.Printf("用户年龄累加结果: %d\n", sum)
}

```
- 写得更通用，就用泛型
- 不过分开调用 Map、Reduce、Filter 函数不太优雅，我们可以通过装饰器模式将它们层层嵌套起来，或者通过管道模式（Pipeline）让这个调用逻辑可读性更好，更优雅

# 流式函数调用

- 常见的流式函数调用（水流一样，在面向对象编程中对应的是流接口模式，可以实现链式处理）。通过组合方式（管道）构建更加复杂的业务功能
- 实现流式调用，前提函数的所需参数,要在总入口先理清
- 将其第二个参数声明为了变长参数类型，表示支持传递多个处理函数，这些处理器函数按照声明的先后顺序依次调用
- 管道技术在 HTTP 请求处理中间件中也有广泛的应用
  
**例子**
``` go
package main

import (
    "log"
)
// 引入了一个 user 结构体替代字典类型，让代码更加简洁，可读性更好
type user struct {
    name string
    age  int
}

// 将 Filter 和 Map 函数中的闭包函数(函数参数)取消掉了，改为直接在代码中实现（先不写通用），以便精简代码
func filterAge(users []user) interface{} {
    var slice []user
    for _, u := range users {
        if u.age >= 18 && u.age <= 35 {
            slice = append(slice, u)
        }
    }
    return slice
}

// 返回值声明成了空接口 interface{} 表示可以返回任何类型。
// 运行时动态对它们的返回值类型做检测，并赋值给指定变量，以便程序可以按照我们期望的路径执行下去
func mapAgeToSlice(users []user) interface{} {
    var slice []int
    for _, u := range users {
        slice = append(slice, u.age)
    }
    return slice
}
// 实现流式调用，前提 函数的参数都是一样的类型 []user
// 将其第二个参数声明为了变长参数类型，表示支持传递多个处理函数，这些处理器函数按照声明的先后顺序依次调用
func sumAge(users []user, pipes ...func([]user) interface{}) int {
    var ages []int
    var sum int
    for _, f := range pipes {
        // 这里是函数 所需要的参数
        result := f(users)
        // 返回值类型检测
        switch result.(type) {
        case []user:
            users = result.([]user)
        case []int:
            ages = result.([]int)
        }
    }
    if len(ages) == 0 {
        log.Fatalln("没有在管道中加入 mapAgeToSlice 方法")
    }
    for _, age := range ages {
        sum += age
    }
    return sum
}

func main() {
    var users = []user{
        {
            name: "张三",
            age: 18,
        },
        {
            name: "李四",
            age: 22,
        },
        {
            name: "王五",
            age: 20,
        },
        {
            name: "赵六",
            age: -10,
        },
        {
            name: "孙七",
            age: 60,
        },
        {
            name: "周八",
            age: 10,
        },
    }
    // 流式调用
    sum := sumAge(users, filterAge, mapAgeToSlice)
    log.Printf("用户年龄累加结果: %d\n", sum)
}
```

sum := sumAge(users, filterAge, mapAgeToSlice)
**如果后面的两个函数调用顺序反了，结果会出问题?**

方案：如果担心人为调用顺序出错的话，可以在sumAge上套一层用于规范调用顺序的，但这样的话，管道就不能自由定义顺序了