---
title: "7.0-go函数" #标题
date: 2023-07-14T16:34:24+08:00 #创建时间
lastmod: 2023-07-14T16:34:24+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- go基础
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

# 一等公民
- 函数本身可以作为值进行传递（参数和返回值）,支持匿名函数和闭包（closure）,函数可以满足接口
  - 函数类型：普通的带有名字的函数 ,匿名函数, 类方法
  - <font color="red"> **函数名的本质就是一个指向其函数内存地址的指针常量**</font>, 可以复制给变量（函数变量），然后变量直接调用


# 声明
- 函数的基本组成为：关键字func、函数名、参数列表、返回值、函数体和返回语句
  - 参数列表、返回值的生命周期是函数被调用时开始
- 编译型语言，所以函数编写的顺序是无关紧要的
- 函数变量:**变量是函数类型**把函数作为值保存到变量中（指向函数名或者匿名函数） 
``` go
func TestFunc(t *testing.T) {
	f1 := func(i int) int { return 1 }
	c := f1(5)              // 函数变量直接调用
	fmt.Printf("c: %+v", c) // c: 1
	// tag: f1 f2指向的值(函数内存地址)一样 +v不好打印 指向函数名或者匿名函数
	fmt.Printf("f1: %T, 内存地址：%p", f1, &f1) // func(int) int, 内存地址：0x1400005c060f2
	f2 := f1
	fmt.Printf("f2: %T, 内存地址：%p", f2, &f2) // func(int) int, 内存地址：0x1400005c068
}
```

## 返回值
- return 语句可以带有零个或多个参数，这些参数将作为返回值供调用者使用，简单的 return 语句也可以用来结束 for 的死循环，或者结束一个协程（goroutine）
- 使用多返回值中的最后一个返回参数,一般返回函数执行中可能发生的错误error
- 多个返回值类型括起来，用逗号分隔每个返回值的类型，使用 return 语句返回时，值列表的顺序需要与函数声明的返回值类型一致
- 可以无返回值 返回值列表的括号是可以省略的
- 同一种类型返回值  (int, int)
- <font color="red">命名返回值 (a, b int)命名的返回值变量的默认值为类型的默认值</font>，即数值为 0，字符串为空字符串，布尔为 false、指针为 nil 等

## 可变参数 变长参数
变长参数指的是函数参数的数量不确定，可以按照需要传递任意数量的参数到指定函数
- <font color="red">...type 语法糖</font>形如 ...type 格式的类型<font color="red">只能作为函数的参数类型存在，**并且必须是函数的最后一个参数**</font>, ...type本质上是一个数组切片，也就是[]type
- 在参数类型前加上 ...前缀，就可以将该参数声明为变长参 调用传参时 后缀...
- 场景 选项模式，可选参数
``` go
// 可变参数类型约束为 int，如果你希望传任意类型，可以指定类型为 interface{}
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
- <font color="red">没有默认参数值</font>
- 引用类型不能直接用nil 要用对应类型的<nil>

## 值传参
函数接收到传递进来的参数后，会将参数值拷贝给声明该参数的变量（也叫形式参数，简称形参）
``` go
type Data struct {
	complax  []int     // 测试切片在参数传递中的效果
	instance InnerData // 实例分配的innerData
	// 注意一下，下面的ptr的值，获取的是地址，具体地址里的值是什么，并不关心
	ptr *InnerData // 将ptr声明为InnerData的指针类型
}

// 代表各种结构体字段
type InnerData struct {
	A int
}

// 声明一个全局变量
var in = Data{
	// 测试切片在参数传递中的效果
	complax: []int{2, 3, 4},
	// 实例分配的innerData
	instance: InnerData{A: 4},
	// 将ptr声明为InnerData的指针类型
	ptr: &InnerData{A: 6},
}

// tag: 证明是值拷贝
func passByValue(inFunc Data) Data {
	fmt.Printf("在函数内部, 打印结构体Data实例的值:   %+v", inFunc)                  // {complax:[2 3 4] instance:{A:4} ptr:0x104ecc358}
	fmt.Printf("在函数内部, 打印结构体Data实例中instance的地址: %p", &inFunc.instance) // 0x1400010a528
	fmt.Printf("在函数内部, 打印结构体Data实例的地址: %p", &inFunc)                   // 0x1400010a510 传参的变量也是值拷贝 形参地址不一样
	return inFunc
}

// 测试，结构体作为函数参数传入
func TestPassByValue(t *testing.T) {
	fmt.Printf("传入函数前, 打印结构体Data实例的值:   %+v", in)                  // {complax:[2 3 4] instance:{A:4} ptr:0x10101c358}
	fmt.Printf("传入函数前, 打印结构体Data实例中instance的地址: %p", &in.instance) // 0x1010308f8
	fmt.Printf("传入函数前, 打印结构体Data实例的地址: %p", &in)                   // 0x1010308e0
	fmt.Println()
	out := passByValue(in) // 传参和返回都是实例
	fmt.Println()
	fmt.Printf("经过函数处理后, 打印结构体Data实例的值: %+v", out)                      // {complax:[2 3 4] instance:{A:4} ptr:0x10101c358}
	fmt.Printf("经过函数处理后, 打印结构体Data实例中instance的地址:   %p", &out.instance) //  0x1400010a4f8 tag：所以非指针的是结构体值直接拷贝
	fmt.Printf("经过函数处理后, 打印结构体Data实例的地址:  %p", &out)                    // 0x1400010a4e0 返回的变量也是值拷贝
}
```
## 引用传参 形参 类型要申明 指针类型 *T
传递给函数的参数是一个指针，而指针代表的是实参的内存地址，修改指针引用的值即修改变量内存地址中存储的值，所以实参的值也会被修改（这种情况下，<font color="red">传递的是变量地址值的拷贝</font>，所以从本质上来说还是按值传参）：

- 传参都是复制语义（值传递和引用传递）
``` go
type People struct {
	Name string
	Age  int8
}

func passTest(p People) {
	fmt.Printf("传递到函数中的地址: %p\n", &p)
	// 传递到函数中的地址: 0xc0000b6060
}

func TestPeople(t *testing.T) {
	myName := People{Name: "renshanwen", Age: 18}
	fmt.Printf("主函数初始化的变量地址: %p\n", &myName) // 0xc0000b6048

	passTest(myName)
	fmt.Printf("不会影响到主函数的变量: %p\n", &myName) // 0xc0000b6048

}

// 指针才能修改值
func passTest2(p *People) {
	fmt.Printf("传递到函数中的指针变量的内存地址: %p，指针变量指向的内存地址：%p\n", &p, p)
	//传递到函数中的指针变量的内存地址: 0xc00000e050，指针变量指向的内存地址：0xc00000c060
	// tag: 证明p  形参p也是直接拷贝指针地址（0xc00000e048），本身是局部变量地址 0xc00000e050
	p.Name = "chengcheng" // 修改名字
}

func TestPeople2(t *testing.T) {
	myName := &People{Name: "renshanwen", Age: 18}
	fmt.Printf("主函数初始化指针变量的内存地址: %p, 指针变量指向的内存地址:%p\n", &myName, myName) // 0xc00000e048 0xc00000c060
	passTest2(myName)
	fmt.Printf("被修改后的名字: %s\n", myName.Name)                              // chengcheng
	fmt.Printf("主函数被影响后指针变量的内存地址: %p, 指针变量指向的内存地址:%p\n", &myName, myName) // 0xc00000e048 0xc00000c060
}

```

``` go
type Person struct {
	Name string `json:"name"`
	Age  *int   `json:"age"`
}

func TestParamPoint(t *testing.T) {
	var p *Person                  // 同样是传参数 如果先初始化后 不同的结果
	fmt.Printf("%p \n", p)         // 0x0
	F(p)                           // 此时传的参数是nil
	fmt.Printf("%+v, %p \n", p, p) // <nil>, 0x0
}

func TestParamPoint2(t *testing.T) {
	var p *Person = &Person{}      // tag: 指针可以修改指向的值
	fmt.Printf("%p \n", p)         // 0x1400000c0c0
	F(p)                           // 此时传的参数是已经分配内存的Person类型指针
	fmt.Printf("%+v, %p \n", p, p) // &{Name: Age:<nil>}, 0x1400000c0c0
	F2(p)
	fmt.Printf("%+v, %p \n", p, p) // &{Name:abc Age:0x1400000e2e0}, 0x1400000c0c0
}

// tag: 参数是复制语义
func F(p1 *Person) *Person {
	// tag: 赋值给的额是 p1这个copy的
	tmp := new(int)
	*tmp = 10
	p1 = &Person{
		Name: "abc",
		Age:  tmp,
	}
	return p1
}

func F2(p1 *Person) *Person {
	p1.Name = "abc"
	var tmp = 100
	p1.Age = &tmp
	return p1
}
```

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

# 闭包
- 函数可以赋值给变量，也可以作为参数传递给其他函数，还能够被函数动态创建和返回
- <font color="red">有状态的匿名函数, 匿名函数引用了外部变量，就形成了一个闭包（Closure）</font>

- **<font color="red">编译器 捕获上层变量的引用，让外部变量从栈逃逸到堆上，然后外部变量跟返回函数一起打包返回</font>**
``` go
func NewIDGenerator() func() int {
	n := 1 // tag: 外部变量 返回匿名函数
	return func() int {
		var base = 100
		n++
		return base + n
	}
}

func f() {
	idGen := NewIDGenerator()
	fmt.Println(idGen()) // 102
	fmt.Println(idGen()) // 103
	fmt.Println(idGen()) // 104
}

// 模拟打包过程
// tag: 打包的外部变量，可以多个变量
type Closure struct {
	n *int
}

// tag: 打包的匿名函数
func (c *Closure) Call() int {
	var base = 100
	(*c.n)++
	return base + *c.n
}

func NewIDGenerator2() *Closure {
	n := 1
	return &Closure{&n}
}

func f2() {
	idGen := NewIDGenerator2()
	fmt.Println(idGen.Call()) // 102
	fmt.Println(idGen.Call()) // 103
	fmt.Println(idGen.Call()) // 104
}

// 并发的时候非安全的 所以使用闭包时需要相关并发原语处理
func Test_NewIDGenerator_Concurrent(
	t *testing.T) {
	idGen := NewIDGenerator()
	const N = 3
	results := make(chan int, N)
	for range N {
		go func() {
			results <- idGen()
		}()
	}
	for range N {
		<-results
	}
}
```
- <font color="red">为什么模拟要使用*int 指针? 因为闭包共享同一上层变量</font>
``` go
// 返回2个闭包的情况，共同使用了变量n 此时闭包捕获的时变量n的引用
func NewIDGenerators(start int) (
	x1 func() int,
	x10 func() int,
) {
	n := start
	x1 = func() (id int) {
		id = n
		n++
		return
	}
	x10 = func() (id int) {
		id = n
		n += 10
		return
	}
	return
}

func f() {
	idGen1, idGen10 :=
		NewIDGenerators(1)
	fmt.Println(idGen1()) // 1
	fmt.Println(idGen1()) // 2

	fmt.Println(idGen10()) // 3  是否想着1
	fmt.Println(idGen10()) // 13 是否想着11
}
// 模拟打包
type Closure1 struct {
	n *int
}
func (c *Closure1) Call() (id int) {
	id = *c.n
	(*c.n)++
	return
}

type Closure10 struct {
	n *int
}
func (c *Closure10) Call() (id int) {
	id = *c.n
	(*c.n) += 10
	return
}

func NewIDGenerators2(start int) (
	*Closure1,
	*Closure10,
) {
	n := start
    // tag: 传的就得时变量n的引用 也就是指针
	return &Closure1{&n}, &Closure10{&n}
}

func main2() {
	idGen1, idGen10 :=
		NewIDGenerators2(1)
	fmt.Println(idGen1.Call()) // 1
	fmt.Println(idGen1.Call()) // 2

	fmt.Println(idGen10.Call()) // 3 // tag:如果打包的是上层变量的值 那么就是1和11
	fmt.Println(idGen10.Call()) // 13
}
```

## 函数返回值为闭包的情况
**<font color="red">通过将函数返回值声明为函数类型来实现业务逻辑的延迟执行, 这里只有真正运行 返回的匿名函数，才真正执行操作,让执行时机完全掌握在开发者手中</font>**
``` go
// adder 函数返回闭包
func adder() func(int) int {
	sum := -1 // 自由变量
	fmt.Printf("adder %d", sum)
	return func(v int) int {
		fmt.Printf("+ %d ", v)
		sum += v // 引用了外部变量 所以是闭包 对应这个闭包形参是调用的时候传入
		fmt.Printf("sum %d ", sum)
		return sum
	}
}

// 闭包 函数返回的是闭包
func TestBb(t *testing.T) {
	a := adder() // 返回了个闭包（打包了匿名函数和上层变量引用）
	for i := 0; i < 10; i++ {
		fmt.Println("cur:", a(i)) // 此时返回闭包调用的结果
	}

	// adder -1+ 0 sum -1 cur: -1
	// + 1 sum 0 cur: 0
	// + 2 sum 2 cur: 2
	// + 3 sum 5 cur: 5
	// + 4 sum 9 cur: 9
	// + 5 sum 14 cur: 14
	// + 6 sum 20 cur: 20
	// + 7 sum 27 cur: 27
	// + 8 sum 35 cur: 35
}
```

## 函数参数为闭包的情况

- <font color="red">增强函数实现装饰器模式(闭包)</font>
- 函数的参数和返回值签名都要与要加强的一致, 并使通过函数创建一个闭包（闭包的参数为要执行的函数）
``` go
//  handleFunc 对应的是装饰器模式中的核心类
type handleFunc func(ctx context.Context, param map[string]interface{}) error

// Decorate 增强方法对应的则是装饰器类
func Decorate(fn handleFunc) handleFunc {
    // 每次在执行 Decorate 的过程中，都会在 handleFunc 前后增加的一些额外的附属逻辑.
    return func(ctx context.Context, param map[string]interface{}) error {
        // ...前处理逻辑
        fmt.Println("preprocess...")

        err := fn(ctx, param)
        // ...后处理逻辑
        fmt.Println("postprocess...")
        return err
    }
}
``` 
- 实现案例
``` go
// timeSpent在这个闭包函数中传入的是一个函数 inner 的类型是func(opt int) int 同时返回这个闭包
// type handleFunc func(op int) int
func timeSpent(inner func(op int) int) func(op int) int {
    // 输入和输出同样的函数类型 实现装饰器模式
    return func(n int) int {
		start := time.Now()
		ret := inner(n) // 传参的真正执行 此时上层变量是inner（函数类型） 
		fmt.Println(ret)
		fmt.Println("time spent:", time.Since(start).Seconds()) // 函数执行耗时
		return ret + 10000
	}
}

func slowFun(op int) int {
	fmt.Println("op:", op)
	time.Sleep(time.Second * 3)
	return op + 1000
}

func TestFn(t *testing.T) {
	// slowFun 函数变量 赋值给新变量 然后再用来传参 等价于直接写函数名
	// var funcVar func(op int) int
	// funcVar = slowFun          //一般是是会用匿名函数
	// tsSF := timeSpent(funcVar) //slowFun对应的参数和返回值 要符合 timeSpent的声明

	// 函数作为参数 以及函数作为返回值 闭包 10也是slowFun参数
	tsSF := timeSpent(slowFun)
	t.Log(tsSF(10)) 

    // op: 10
    // 1010
    // time spent: 3.001072083
}

```
- **为了实现更加通用的函数执行耗时计算功能，应该将 timeSpent 函数参数和返回值声明为泛型**
  
## 用处场景
- 匿名函数内部声明的局部变量无法从外部修改（类的构造，保护私有成员）
- (高阶函数)将匿名函数<font color="red">作为函数参数</font>, Go 官方 net/http 包底层的路由处理器也是这么实现的（适配器模式）
- (高阶函数)将匿名函数<font color="red">作为函数返回值</font> （迭代器模式）

# <font color="red">方法就是函数</font>
``` go
- 方法本质上就是函数的**语法糖**,为了更清晰地表达面向对象的设计思想而提供的一种便捷写法
  - 方法与函数的区别在于，方法拥有接收者，而函数没有，且只有自定义类型能够拥有方法
  - 方法其实是可以通过 类型.方法(接受者, 其他方法参数...)，就像使用函数一样使用方法
type Student struct {
	Name string
	Age  int
}

// 绑定到结构体指针 为了方便修改对应的实体属性
func (s *Student) Say(content string) string {
	fmt.Println(s.Name, s.Age, content)
	return content
}

// 绑定到结构体
func (s Student) Hi() {
	fmt.Println("hello world")
}

func TestFunc(t *testing.T) {
	s := Student{Name: "张三", Age: 18}
	// 第一种方式：正常调用
	s.Say("a") // 张三 18 a
	// 第二种方式：使用类型名调用
	// 类型是Student
	Student.Hi(s) // hello world
    (*Student).Hi(&s) // hello world 指针类型也是实现了的
	// 是指针类型 类型是*Student
	res1 := (*Student).Say(&s, "b") //  张三 18 b
	fmt.Println(res1)               // b

	// 第三种方式：使用变量名调用 函数名本质（一等公民）
	var sSay = s.Say  // var sSay func(content string) string = s.Say
	res2 := sSay("c") // 张三 18 c
	fmt.Println(res2) // c
}
```

# 以下函数式编程技术

# 高阶函数
高阶函数，就是接收其他函数作为参数传入，或者把其他函数作为结果返回的函数

## 实现装饰器模式（函数为结果）
{{< innerlink src="posts/tech/go9.md" >}} 

- 增强函数写法，装饰器模式 中间件

## 实现适配器模式（函数为参数）
{{< innerlink src="posts/tech/go30.md" >}}  

- 配合适配器模式: interface里定义方法。额外实现一个适配器(实现了interface)，对函数变量进行转化
- 增强函数写法 将函数（func）转换为接口（type）优点 注入自定义闭包函数
## 实现装选项模式模式
{{< innerlink src="posts/tech/go23.md" >}} 

- 函数选项模式option 结构体的创建

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
- https://go-zero.dev/zh-cn/components/concurrency/mr/ 开源的包 执行 MapReduce 操作的框架

- 写得更通用，就用泛型
- 不过分开调用 Map、Reduce、Filter 函数不太优雅，<font color="red">**我们可以通过装饰器模式将它们层层嵌套起来，或者通过管道模式（Pipeline）让这个调用逻辑可读性更好，更优雅**</font>

## Stream流式处理
- https://go-zero.dev/zh-cn/components/concurrency/fx/ 开源的包 用于流处理的函数和类型
  - 常见的流式函数调用（水流一样，在面向对象编程中对应的是流接口模式，可以实现链式处理）。通过组合方式（管道）构建更加复杂的业务功能
  - <font color="red">实现流式调用，前提函数的所需参数,要在总入口先理清</font>
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

- 管道模式（Pipeline） <font color="red">实现流式调用，前提 函数的参数都是一样的类型</font> []user
- <font color="red">将参数声明为了变长参数类型</font>，表示支持传递多个处理函数，这些处理器函数按照声明的先后顺序依次调用
- **如果后面的两个函数调用顺序反了，结果会出问题?**
  - sum := sumAge(users, filterAge, mapAgeToSlice)
  - 方案：如果担心人为调用顺序出错的话，可以在sumAge上套一层用于规范调用顺序的，但这样的话，管道就不能自由定义顺序了

## 配合interface实现

{{< innerlink src="posts/tech/go23.md" >}}  

- 链式调用，函数接收器，使其返回修改后的对象本身，即使它们通常不返回任何内容, 建造者模式。区别管道模式

# 嵌套集合模型
- 递归，平铺循环，嵌套集合模型性能差
<font color="red">[【有道云笔记】go树形结构实现和生成性能比较和嵌套集合](https://note.youdao.com/s/dPUvZplB)</font>
