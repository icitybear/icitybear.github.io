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
# 原理
- 装饰器模式和“继承”一定的类似之处，但是两者侧重点有所不同，可以把装饰器模式作为“继承”的一种补充手段.
- 通过**高阶函数**实现(增强函数实现), <font color="red">实现也是遵循着装饰器模式的思路，但在形式上会更加简洁直观一些</font>
- 本身是在强依附于核心类（主食）的基础上产生的，只能起到锦上添花的作用，因此在构造器函数中，需要传入对应的主食 Food（核心类接口）

## 继承与装饰器区别
- 继承强调的是等级制度和子类种类，这部分架构需要一开始就明确好
- <font color="red">装饰器模式强调的是“装饰”的过程，而不强调输入与输出，能够动态地为对象增加某种特定的附属能力，相比于继承模式显得更加灵活，且符合开闭原则</font>
## 演进
对米饭进行加工
• 比如选用主菜培根搭配上副菜鸡蛋，就形成了一碗鸡蛋培根盖浇饭；

• 比如选用主菜牛肉搭配上副菜青椒，就形成了一碗青椒牛肉盖浇饭；

• 比如选用主菜鸡排搭配上副菜黑椒汁，就形成了一碗黑椒鸡排盖浇饭.
### 继承的方式进行实现
![alt text](image1.png)
继承”的实现思路中，我们需要对子类的等级以及种类进行枚举，包括通过加入主菜后形成的一系列一级子类以及加入主菜和副菜后形成的一系列二级子类
- <font color="red">缺点：</font>
  - 主菜和副菜的结合可以是更加灵活多样的, 后续可能有更多的主菜和副菜类型出现,对配菜的类型进行界定显得过于刻板，主菜和副菜本质上都是菜品而已，可以根据用户的喜好灵活添加，比如用户可以只要副菜或者只要主菜
  - 对配菜的类型进行界定显得过于刻板，主菜和副菜本质上都是菜品而已，可以根据用户的喜好灵活添加，比如用户可以只要副菜或者只要主菜

### 装饰器模式
- <font color="red">不再聚焦于尝试对所有组合种类进行一一枚举，而是把注意力放在“加料”的这个过程行为当中</font>
![alt text](image2.png)
一份鸡蛋培根盖浇饭 = 一份白米饭（核心类）+ 一份鸡蛋（装饰器1）+ 一份培根（装饰器2），<font color="red">其中鸡蛋和培根对应装饰器的使用顺序是不作限制的.</font>于是不管后续有多少种新的“菜品”加入，我们都只需要声明其对应的装饰器类即可

# 代码实现
主食包括米饭 rice 和面条 noodle 两条，而配菜则包括老干妈 LaoGanMa（老干妈拌饭顶呱呱）、火腿肠 HamSausage 和煎蛋 FriedEgg 三类.
![alt text](image3.png)
## 主食类
``` go
type Food interface {
    // 食用主食
    Eat() string
    // 计算主食的花费
    Cost() float32
}

// 主食-米饭
type Rice struct {
}
func NewRice() Food {
    return &Rice{}
}
func (r *Rice) Eat() string {
    return "开动了，一碗香喷喷的米饭..."
}
// 需要花费的金额
func (r *Rice) Cost() float32 {
    return 1
}

// 主食-面条
type Noodle struct {
}
func NewNoodle() Food {
    return &Noodle{}
}
func (n *Noodle) Eat() string {
    return "嗦面ing...嗦~"
}
// 需要花费的金额
func (n *Noodle) Cost() float32 {
    return 1.5
}
```
## 装饰器
``` go
// 装饰器部分，我们声明了一个 Decorate interface，它们本身是在强依附于核心类（主食）的基础上产生的，只能起到锦上添花的作用，因此在构造器函数中，需要传入对应的主食 Food.
type Decorator Food // 装饰器类型 都是依赖要装饰的核心类上
func NewDecorator(f Food) Decorator {
    return f // 因此在构造器函数中，需要传入对应的主食 Food.
}

// 每个装饰器类的作用是对食物进行一轮装饰增强，因此需要在构造器函数中传入待装饰的食物，然后通过重写食物的 Eat 和 Cost 方法，实现对应的增强装饰效果.'
type LaoGanMaDecorator struct {
    Decorator // 都继承基础的装饰器类型
}
func NewLaoGanMaDecorator(d Decorator) Decorator {
    return &LaoGanMaDecorator{
        Decorator: d,
    }
}
func (l *LaoGanMaDecorator) Eat() string {
    // 加入老干妈配料
    return "加入一份老干妈~..." + l.Decorator.Eat()
}
func (l *LaoGanMaDecorator) Cost() float32 {
    // 价格增加 0.5 元
    return 0.5 + l.Decorator.Cost()
}

type HamSausageDecorator struct {
    Decorator
}
func NewHamSausageDecorator(d Decorator) Decorator {
    return &HamSausageDecorator{
        Decorator: d,
    }
}
func (h *HamSausageDecorator) Eat() string {
    // 加入火腿肠配料
    return "加入一份火腿~..." + h.Decorator.Eat()
}
func (h *HamSausageDecorator) Cost() float32 {
    // 价格增加 1.5 元
    return 1.5 + h.Decorator.Cost()
}

// FriedEggDecorator 省略 。。。
```
## 测试代码
``` go
func Test_decorator(t *testing.T) {
    // 核心类
    // 一碗干净的米饭
    rice := NewRice()
    rice.Eat()
    // 一碗干净的面条
    noodle := NewNoodle()
    noodle.Eat()

    // 装饰器 核心类作为构造器参数
    // 米饭加个煎蛋
    rice = NewFriedEggDecorator(rice)
    rice.Eat()
    // 面条加份火腿
    noodle = NewHamSausageDecorator(noodle)
    noodle.Eat()
    // 米饭再分别加个煎蛋和一份老干妈
    rice = NewFriedEggDecorator(rice)
    rice = NewLaoGanMaDecorator(rice)
    rice.Eat()
}
``` 
# <font color="red">增强函数实现装饰器模式(闭包)</font>
- 拦截前后会执行的核心逻辑执行方法（核心类）type xxx func(xxx)xx
- 函数的参数和返回值签名都要与要加强的一致
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

// 以下具体调用  再进一步是中间件写法
```

## 小例子-包装耗时功能(闭包)
- 通过**高阶函数**实现
- 核心思路就是在被修饰的功能模块（这里是外部传入的乘法函数 f）**<font color="red">执行前后加上一些额外的业务逻辑，而又不影响原有功能模块的执行.</font>**
- 在 main 函数中调用乘法函数 multiply 时，如果要应用装饰器，需要通过装饰器 execTime 包裹，装饰器返回的是个匿名函数，所以需要再度调用<font color="red">(延时执行)</font>才能真正执行，
### 基本功能模块
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

### 装饰器模式增强
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
- **为了实现更加通用的函数执行耗时计算功能，应该将 MultiPlyFunc 函数参数和返回值声明为泛型**

# grpc-go 中对拦截器链(Interceptor) chainUnaryInterceptors 的实现（闭包）
https://github.com/grpc/grpc-go
![alt text](image4.png)
- 通过装饰器模式，生成拦截器Interceptor

每次接收到来自客户端的 grpc 请求，会根据请求的 path 映射到对应的 service 和 handler 进行执行逻辑的处理，但在真正调用 handler 之前，会先先经历一轮对拦截器链 chainUnaryInterceptors 的遍历调用

核心业务处理方法 handler相当于核心类（就是UnaryHandler），每一轮通过拦截器 UnaryServerInterceptor 对 handler 进行增强的过程，对应的就是一次“装饰”的步骤.
``` go

// 拦截前后 会执行的核心逻辑执行方法（核心类）  入参为 context 和 req，出参为 resp 和 error
type UnaryHandler func(ctx context.Context, req interface{}) (interface{}, error)

// • req：grpc 请求的入参
// • info：grpc 业务服务 service
// • handler：核心逻辑执行方法 UnaryHandler
type UnaryServerInterceptor func(ctx context.Context, req interface{}, info *UnaryServerInfo, handler UnaryHandler) (resp interface{}, err error)

// 增强函数模式 （闭包）-装饰器模式-实现拦截器链
func chainUnaryInterceptors(interceptors []UnaryServerInterceptor) UnaryServerInterceptor {
    return func(ctx context.Context, req interface{}, info *UnaryServerInfo, handler UnaryHandler) (interface{}, error) {
        // 会调用拦截器列表 interceptors 当中的首个拦截器 并(调用getChainUnaryHandler返回的类型UnaryHandler)
        return interceptors[0](ctx, req, info, getChainUnaryHandler(interceptors, 0, info, handler))
        // 最终执行的核心类
    }
}

// 增强函数模式 （闭包）实现装饰器模式-生成拦截器
// 依次使用下一枚拦截器对核心方法 handler 进行装饰包裹，封装形成一个新的“handler”供当前的拦截器使用.
func getChainUnaryHandler(interceptors []UnaryServerInterceptor, curr int, info *UnaryServerInfo, finalHandler UnaryHandler) UnaryHandler {
    if curr == len(interceptors)-1 {
        return finalHandler
    }
    return func(ctx context.Context, req interface{}) (interface{}, error) {
        return interceptors[curr+1](ctx, req, info, getChainUnaryHandler(interceptors, curr+1, info, finalHandler))
    }
}

// demo
var myInterceptor = func(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (resp interface{}, err error) {
    // 添加前处理...
    fmt.Printf("interceptor preprocess, req: %+v\n", req)
    resp, err = handler(ctx, req)
    // 添加后处理...
    fmt.Printf("interceptor postprocess, req: %+v\n", resp)
    return
}
// 将myInterceptor拦截器加入拦截器链路里
```
- 比如[流量控制sentinel的接入](https://pkg.go.dev/github.com/alibaba/sentinel-golang/pkg/adapters/grpc)