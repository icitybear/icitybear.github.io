---
title: "4.0-go的复合类型-map字典和列表包container/list" #标题
date: 2023-07-14T11:20:41+08:00 #创建时间
lastmod: 2023-07-14T11:20:41+08:00 #更新时间
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

- <font color="red">[文章-Golang map 实现原理](https://mp.weixin.qq.com/s/PT1zpv3bvJiIJweN3mvX7g)</font>

# 声明和初始化
- 关键字map
- Go 字典也是个<font color="red">无序集合</font>，底层不会按照元素添加顺序维护元素的存储顺序
- map 是引用类型, map 这种数据结构在其他编程语言中也称为字典（Python）、hash 和 HashTable 等。
- 声明 var name <font color="red">map[keytype]valuetype</font> , keytype键的类型，valuetype所存放的值类型。初始化{}和赋值便捷写法都支持
- make创建并初始化，<font color="red">可以使用 make()</font>，但不能使用 new() 来构造 map，
  - <font color="red">make(map[keytype]valuetype, cap) 此时第二个参数为容量与make([]Type, size, cap ) 切片有所区别</font>
  - 如果错误的使用 new() 分配了一个引用对象，会获得一个空引用的指针，相当于声明了一个未初始化的变量并且取了它的地址  new返回指向类型的指针 通（*mv）= make(xxx, xx)再出分配内存初始化
  - <font color="red">make方式初始化后，一开始为空map (map[]), 可以往字典中添加键值对（前面那种var声明方式不能这么操作，否则编译期间会抛出 panic, testMap["one"] = 1</font>
- 容量超出会自动扩容

# 元素赋值
- <font color="red">字典初始化之后才能进行赋值操作</font>, 仅仅是声明var，此时 testMap 的值为 nil，在 nil 上进行操作编译期间会报 panic（运行时恐慌 panic: assignment to entry in nil map），导致编译不通过。
- 应该使用make或者{},声明且初始化
  
``` go
func TestInitMap(t *testing.T) {
    // 错误的
	var m0 map[int]string
	m0[0] = "hello" // panic: assignment to entry in nil map [recovered]
	t.Log(m0)
    // 正确的
	m1 := map[int]int{1: 1, 2: 4, 3: 9}
	t.Log(m1[2])
	t.Logf("len m1=%d", len(m1))
    // 正确的
	m2 := map[int]int{}
	m2[4] = 16
	t.Logf("len m2=%d", len(m2))
	//正确的
	m4 := make(map[int]string, 10)
	t.Logf("len m4=%d", len(m3))
	m4[0] = "hello"
	t.Log(m4) // map[0:hello]
}
```
# 查找元素
- 从字典中查找指定键时，会返回两个值，第一个是真正返回的键值，第二个是是否找到的标识,布尔值。 配合匿名变量，也可以省略
- 如果未找到值为默认类型零值，ok == false
``` go
value, ok := testMap["one"] 
if ok { // 找到了
  // 处理找到的value 
}

value, _ := testMap["one"] 

value := testMap["one"]
```
- 底层也是通过哈希表实现的，**<font color="red">添加键值对到字典时，实际是将键转化为哈希值进行存储，(自动构建哈希值)</font>**, 在查找时，也是先将键转化为哈希值去哈希表中查询，从而提高性能。
  - 哈希冲突  Go 底层还会判断原始键的值是否相等. 所以声明字典的键类型时，要求数据类型必须是<font color="red">支持通过 == 或 != 进行判等操作的类型</font>,比如数字类型、字符串类型、数组类型、结构体类型等, 非动态类型、非指针、函数、闭包。
  - 提高字典查询性能，类型长度越短越好

# 删除元素 delete()通过键名
delete(testMap, "four")
- 从 testMap 中删除键为「four」的键值对。如果「four」**<font color="red">这个键不存在或者字典尚未初始化，这个调用也不会有什么副作用</font>**
- 删除后长度也是动态减的
- <font color="red">清空 map 的唯一办法就是重新 make 一个新的 map</font>，不用担心垃圾回收的效率，Go语言中的并行垃圾回收效率比写一个清空函数要高效的多

# 遍历 range
- 由于字典是无序的, 获取的时候也是无序的, 可以配合匿名变量_
- 值也是副本

``` go
testMap := map[string]int{
    "one": 1,
    "two": 2,
    "three": 3,
}
for key, value := range testMap {
    fmt.Println(key, value)  //输出时无序 three 3 one 1 two 2
}

// 配合匿名变量_
for _, value := range testMap {
    fmt.Println(value)
}

// 只获取字典的键名
for key := range testMap {
    fmt.Println(key)
}

```

# map排序
- json序列化map类型的时候是无序的
1. 使用切片sort包 正确的做法是 **输入到临时切片里，然后对切片元素排序（要额外调用切片排序）**

2. orderedmap包, 根据切片顺序设置对应键值
``` go
import "github.com/iancoleman/orderedmap"
func TestOrder(t *testing.T) {
	m1 := map[string]int{
		"a":  1,
		"c":  2,
		"b":  3,
		"c1": 4,
	}
	fmt.Println(m1) // 输出的时候key自然顺序 map[a:1 b:3 c:2 c1:4]
	b, _ := json.Marshal(m1)
	fmt.Println(string(b)) // {"a":1,"b":3,"c":2,"c1":4}
	// 序列化也是一样的
	// 1 想要指定顺序 包wrappers "google.golang.org/protobuf/types/known/wrapperspb"
	// 2 使用封装好的
	o := orderedmap.New()
	o.Set("a", 1) // 按照顺序设置
	o.Set("c1", 4)
	o.Set("b", 3)
	o.Set("c", 2)
	keys := o.Keys()
	for _, k := range keys {
		v, _ := o.Get(k)
		fmt.Println(k, v)
	}
	c, _ := json.Marshal(o)
	fmt.Println(string(c))  // {"a":1,"c1":4,"b":3,"c":2}

}

func TestOrderStruct(t *testing.T) {
	result := []MyStruct{
		// 添加结构体实例
		{"name1", 1, "10", "20", "class1", 1744775442},
		{"name2", 2, "10", "20", "class2", 1744775442},
		{"name3", 3, "10", "20", "class3", 1744775442},
	}
	// tag: 最好与结构体字段名一致 不是以json为准 （最多转下大驼峰命名 匹配字段）
	customSortSlice := []string{"name", "age", "avgScore", "SumScore", "class_name", "createdTime"}
	customTitleMap := map[string]string{
		"name":        "名称",
		"age":         "年龄",
		"avgScore":    "平均",
		"SumScore":    "总分",
		"class_name":  "课程名",
		"createdTime": "创建时间",
	}

	tempMap := make([]*orderedmap.OrderedMap, 0)
	for _, item := range result {
		// 使用orderedmap 不然得定义个排序结构体，方便序列化时有序
		o := orderedmap.New()          // o.Set 设置字段标题 跟值 支持map排序的
		targetObj := structs.New(item) // 提供操作 struct, tag, field 的相关函数
		// fmt.Printf("%+v", targetObj)   // 转为对象都是对应的字段名
		// &{
		// raw:{Name:name1 Age:1 AvgScore:10 SumScore:20 ClassName:class1 CreatedTime:1744775442}
		// rtype:0x100b67040
		// rvalue:{typ:0x100b67040 ptr:0x140001009b0 flag:153}
		// TagName:json
		// }

		// 显示个性标题
		for _, dims := range customSortSlice {
			// dims是自己想显示的 key 对应有map设置， 然后映射到结构体对应的字段
			fieldDims := UnderscoreToCamel(dims)
			dimsName := customTitleMap[dims]
			fieldDimsVal, ok := targetObj.Field(fieldDims)
			if ok {
				dimsValue := fieldDimsVal.Value()
				if fieldDims == "CreatedTime" {
					// tag: 类似元素为数组的情况，时间戳转时间字符串的情况
					if createdTime, ok := fieldDimsVal.Value().(int); ok {
						dimsValue = time.Unix(int64(createdTime), 0).In(time.Local).Format("2006-01-02 15:04:05")
					}
				}
				o.Set(dimsName, dimsValue)
			} else {
				o.Set(dimsName, "")
			}
		}

		// for _, targetKey := range sortSlice {
		// 	targetKey2 := util.UnderscoreToCamel(targetKey) // 下划线转驼峰因为结果集是结构体
		// 	targetName := titleMap[targetKey]
		// 	if mField, ok := targetObj.Field(targetKey2); ok {
		// 		o.Set(targetName, mField.Value())
		// 	}
		// }

		tempMap = append(tempMap, o)
	}

	spew.Println(tempMap)
}

```

## 框架返回是，序列化的处理
- 本身序列化map (kv结构) 改为 结构体对象
- 利用google.golang.org/protobuf/types/known/wrappersp
``` go
   tempMap := make([]*orderedmap.OrderedMap, 0)
    for ...{
        o := orderedmap.New()
        o.Set(dimsName, dimsValue)
    }

    m := map[string]interface{}{
        "data": tempMap,
    }
    b, _ := json.Marshal(m) // 本身序列化自己的排序字符串
    resp := &wrapperspb.StringValue{}
    resp.Value = string(b)
    return resp, nil
```

![alt text](map4.png)

# 多维map
- map[int]map[string]string, 二维的情况下 一维的值的类型还是map字典类型
- 如果map未初始化分配内存，要先分配内存

# 空map (声明并分配内存了，只是元素零值)
- 通过make初始化
- 通过{}, m2 := map[string]int{}
- <font color="red">多维map，查找元素时，要用len防止空map</font>
	格式是这样的 **map[int]map[string]string** ,redis处理后是 比如传21 [21 => map[]] 这种空map，如果只是声明应该是  [21 => nil]。 如果一维对应的值此时是一个<font color="red">空map（已经初始化了{} x:= map[string]string{}）</font>, 不能 if _, ok 用ok去判断，这时候 ok==true, 要使用len
![](map1.png)
![](map2.png)

``` go
	result := make(map[string][]string, 0)
	for _, v := range list {
		menuId := strconv.Itoa(v.MenuId)
		if _, ok := result[v.AgentProjectCode]; ok {
			result[v.AgentProjectCode] = append(result[v.AgentProjectCode], menuId)
		} else {
			result[v.AgentProjectCode] = []string{menuId}
		}
	}
	// 优化 直接使用用append
	result := make(map[string][]string, 0)
	for _, v := range list {
		menuId := strconv.Itoa(v.MenuId)
		result[v.AgentProjectCode] = append(result[v.AgentProjectCode], menuId)
	}
```
原因：Go 中 map 访问不存在的 key 会返回值类型的零值，对 []string 来说就是 nil，而 append(nil, elem) 会自动创建新切片。所以 if _, ok 的守卫完全多余

# 多键索引
- [Go语言map的多键索引——多个数值条件可以同时查询](https://note.youdao.com/s/D0OQeLbt)-<font color="red">添加键值对到字典时，实际是将键转化为哈希值进行存储</font>
- 底层会为 map 的键自动构建哈希值。能够构建哈希值的类型必须是非动态类型、非指针、函数、闭包

# 并发不安全
- Go 内置 map 的非原子性操作和底层哈希表结构的复杂性导致的

1. 数据竞争（Data Race）：当多个协程同时对 map 进行读写（例如一个协程写入，另一个协程读取或写入），会导致数据竞争。这种竞争可能引发 panic，例如 fatal error: concurrent map writes 
2. 哈希表内部结构破坏：map 的底层实现基于哈希表，当触发扩容（rehash）时，若并发读写可能破坏内部指针或桶结构，导致程序崩溃或数据丢失
3. 结构体赋值的非原子性：即使是对结构体中的 map 字段赋值（如 struct.MapField = newMap），若未加锁，其他协程可能读取到部分初始化的中间状态
## 场景
1. 协程 A 和 B 同时写入同一个 map：可能导致键值对覆盖或哈希桶损坏。
2. 协程 A 写入，协程 B 读取：可能读到不完整的数据（如扩容过程中的中间状态），或触发 panic。
3. 结构体中的 map 字段并发操作：即使结构体本身是局部变量，若多个协程共享该结构体实例的 map，仍然存在并发风险
## 案例
``` go
// 危险：多个协程并发写入 map
func main() {
    m := make(map[string]int)
    var wg sync.WaitGroup
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            m[fmt.Sprintf("key%d", i)] = i // 触发并发写 panic
        }(i)
    }
    wg.Wait()
}
```

# sync.Map 并发安全的map
**为什么需要这个？**
map 在并发情况下，只读是线程安全的，同时读写是线程不安全的

使用了两个并发函数不断地对 map 进行读和写而发生了竞态问题，fatal error: concurrent map read and map write

**需要并发读写时，一般的做法是加锁，（锁，尽管是读锁也不要乱用   锁冲突），但这样性能并不高**
![](map3.png)
- sync.Map会有两块区域，一个是只读，一个是可读可写，其中可读可写的区域也是有一个锁的。每次操作时，会先去只读区域中寻找，只读区域未找到的时候触发miss，然后才会去可读可写区域寻找
- sync.Map 没有提供获取 map 数量的方法，替代方法是在获取 sync.Map 时遍历自行计算数量

``` go
package main

import (
      "fmt"
      "sync"
)

func main() {

    var scene sync.Map

    // 将键值对保存到sync.Map 值any只能存一种类型，以第一次执行为准
    scene.Store("greece", 97)
    scene.Store("london", 100)
    scene.Store("egypt", 200)

    // 从sync.Map中根据键取值
    fmt.Println(scene.Load("london"))

    // 根据键删除对应的键值对
    scene.Delete("london")

    // 遍历所有sync.Map中的键值对
    scene.Range(func(k, v interface{}) bool {

        fmt.Println("iterate:", k, v)
        return true
    })

}
```

# 分片加锁 concurrent-map包
- https://github.com/orcaman/concurrent-map
- 支持存储任意类型的键和值，只要键是 comparable 可比较的
``` go
// 存储任意类型的示例
m := cmap.New[any]()
m.Set("key1", 123)       // 整型
m.Set("key2", 3.14)      // 浮点型
m.Set("key3", struct{}{}) // 结构体
```

# 并发安全总结
1. 高频读写：分片加锁（如 concurrent-map）。
2. 读多写少：sync.Map 或读写锁。
3. 低频全量替换：atomic.Value (所以也不太合适)
   - atomic.Value：适合原子替换整个对象 
   - sync.Mutex：适合保护操作过程
``` go
import "github.com/orcaman/concurrent-map"
cmap := cmap.New()
cmap.Set("key", "value")

var m sync.Map
m.Store("key", "value")
value, _ := m.Load("key")

var atomicMap atomic.Value
atomicMap.Store(make(map[string]int)) 

type Config struct {
	Addr string
	Port int
}

func main() {
	var config atomic.Value

	// 存储值 (必须同类型)
	cfg := Config{Addr: "127.0.0.1", Port: 8080}
	config.Store(cfg)

	// 加载值 (需类型断言)
	if v := config.Load(); v != nil {
		loadedCfg := v.(Config) // 类型断言
		println(loadedCfg.Addr) // 127.0.0.1
	}
}
```

# 列表 container/list包
- 列表是一种非连续的存储容器，由多个节点组成，节点通过一些变量记录彼此之间的关系，列表有多种实现方法，如单链表、双链表等。
- <font color="red"> 列表使用container/list包来实现，**内部的实现原理是双链表** </font>

## 初始化
- 变量名 := list.New() 和 var 变量名 list.List

列表与切片和 map 不同的是，列表并没有具体元素类型的限制，因此，列表的元素可以是<font color="red">任意类型</font>，这既带来了便利，也引来一些问题，例如给列表中放入了一个 interface{} 类型的值，取出值后，如果要将 interface{} 转换为其他类型将会发生宕机。

## 插入元素
从前方PushFront 和 从后方PushBack，插入元素，都会返回一个 <font color="red">*list.Element 结构</font>，如果在以后的使用中需要**删除插入的元素**，则只能通过 *list.Element 配合 Remove() 方法进行删除，这种方法可以让删除更加效率化，同时也是双链表特性之一。
``` go
l := list.New()
element := l.PushBack(xxx)
l.InsertAfter(xxx, element)
l.InsertBefore(xxx, element)
l.Remove(element)
```

## 删除元素 Remove
 *list.Element 结构，这个结构**记录着列表元素的值以及与其他节点之间的关系等信息**，从列表中删除元素时，需要用到这个结构进行快速删除。配合 Remove() 方法

## 遍历列表 
- 遍历双链表需要配合 Front() 函数获取头元素，遍历时只要元素不为空就可以继续进行，每一次遍历都会调用元素的 Next() 函数

``` go
package main

import "container/list"

func main() {
    l := list.New()

    // 尾部添加
    l.PushBack("canon")

    // 头部添加
    l.PushFront(67)

    // 尾部添加后保存元素句柄
    element := l.PushBack("fist")
    // 后面要操作删除元素
    // 在fist之后添加high
    l.InsertAfter("high", element)

    // 在fist之前添加noon
    l.InsertBefore("noon", element)

    // 使用
    l.Remove(element)
    //使用 for 语句进行遍历，其中 i:=l.Front() 表示初始赋值，只会在一开始执行一次，
    //每次循环会进行一次 i != nil 语句判断，
    //如果返回 false，表示退出循环，反之则会执行 i = i.Next()
    for i := l.Front(); i != nil; i = i.Next() {
        fmt.Println(i.Value)
    }
}
```
对应的结果

|操作内容  | 列表元素|
|:--|:--|
l.PushBack("canon") | canon
l.PushFront(67)| 67, canon
element := l.PushBack("fist")| 67, canon, fist
l.InsertAfter("high", element)| 67, canon, fist, high
l.InsertBefore("noon", element)| 67, canon, noon, fist, high
l.Remove(element)| 67, canon, noon, high