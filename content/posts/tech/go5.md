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

# 声明和初始化
- 关键字map
- Go 字典也是个<font color="red">无序集合</font>，底层不会按照元素添加顺序维护元素的存储顺序
- map 是引用类型, map 这种数据结构在其他编程语言中也称为字典（Python）、hash 和 HashTable 等。
- 声明 var name <font color="red">map[keytype]valuetype</font> , keytype键的类型，valuetype所存放的值类型。初始化{}和赋值便捷写法都支持
- make创建并初始化，<font color="red">可以使用 make()</font>，但不能使用 new() 来构造 map，
  - make(map[keytype]valuetype, cap)  map类型  （与make([]Type, size, cap ) 切片有所区别）
  - 如果错误的使用 new() 分配了一个引用对象，会获得一个空引用的指针，相当于声明了一个未初始化的变量并且取了它的地址  new返回指向类型的指针
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
- 从字典中查找指定键时，会返回两个值，第一个是真正返回的键值，第二个是是否找到的标识,布尔值。 配合匿名变量
``` go
value, ok := testMap["one"] 
if ok { // 找到了
  // 处理找到的value 
}
```
- 底层也是通过哈希表实现的，添加键值对到字典时，实际是将键转化为哈希值进行存储，<font color="red">(自动构建哈希值)</font>, 在查找时，也是先将键转化为哈希值去哈希表中查询，从而提高性能。
  - 哈希冲突  Go 底层还会判断原始键的值是否相等. 所以声明字典的键类型时，要求数据类型必须是<font color="red">支持通过 == 或 != 进行判等操作的类型</font>,比如数字类型、字符串类型、数组类型、结构体类型等,非动态类型、非指针、函数、闭包。
  - 提高字典查询性能，类型长度越短越好

# 删除元素 delete()通过键名
delete(testMap, "four")
- 从 testMap 中删除键为「four」的键值对。如果「four」**这个键不存在或者字典尚未初始化，这个调用也不会有什么副作用**
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

// 进行键值对调 比如PHP 的 array_flip 函数
invMap := make(map[int] string, 3)
for k, v := range testMap {
    invMap[v] = k
}

```

# 排序
- 如果需要特定顺序的遍历结果，正确的做法是 输入到临时切片里，然后对切片元素排序（要额外调用切片排序）
- Go 语言<font color="red">内置的 sort 包</font>，这个包提供了一系列对切片和用户自定义集合进行排序的函数。
  - 对切片进行排序 sort.Ints(values)    sort.Strings(keys) 对键进行排序

``` go
// sort.Sort 自定义排序
// 首先要自定义类型
// 之后要实现自定义排序  需要实现三个方法  Len 返回长度 Less 比较（升序还是降序） Swap 交换
type arrTest []int

func (arr arrTest) Len() int {
	return len(arr)
}

// < 小于号  —— 升序
// > 大于号 —— 降序
func (arr arrTest) Less(i, j int) bool {
	return (arr)[i] > (arr)[j]
}

func (arr arrTest) Swap(i, j int) {
	arr[i], arr[j] = arr[j], arr[i]
	return
}

func TestSliceSort(t *testing.T) {

	//a := []int{12, 45, 33, 78, 9, 14}
	a := arrTest{12, 45, 33, 78, 9, 14}
	sort.Sort(a)
	fmt.Println(a)
}

type person struct { //定义对象person
	name string
	age  int
}

type personSlice []person //给[]person绑定对象

// 实现sort包定义的Interface接口
func (s personSlice) Len() int {
	return len(s)
}

// asc
func (s personSlice) Less(i, j int) bool {
	return s[i].age < s[j].age // 使用切片元素结构体的字段
}

func (s personSlice) Swap(i, j int) {
	s[i].age, s[j].age = s[j].age, s[i].age
}

func TestSliceSort2(t *testing.T) {
	p := personSlice{
		person{
			name: "mike",
			age:  13,
		}, person{
			name: "jane",
			age:  12,
		}, person{
			name: "peter",
			age:  14,
		}}
	sort.Sort(p)
	fmt.Println(p) // [{mike 12} {jane 13} {peter 14}]
}

// sort.Slice(slice, func(i, j int) bool)
// 任意类型slice, 比较函数如果省略这个函数，则使用内置的比较函数对切片进行排序
// 1.8引入
func TestSliceSort3(t *testing.T) {
	s := []int{5, 2, 6, 3, 1, 4}
	sort.Slice(s, func(i, j int) bool {
		// < 小于号  —— 升序
		// > 大于号 —— 降序
		return s[i] < s[j]
	})
	fmt.Println(s)
}
```

# 多维切片
- （一般二维）  map[int]map[string]string, 一维的值的类型还是map字典类型
## 空map
- 通过make初始化
- 通过{}, m2 := map[string]int{}
<font color="red">多维map，查找元素时，要用len防止空map， 业务用时</font>
格式是这样的 **map[int]map[string]string** ,然后redis处理后是 比如传21 [21 => map[]]
如果值此时是一个<font color="red">空map（已经初始化了{} x:= map[string]string{}）</font>,不能 if _, ok 用ok去判断?  得用len
![](map1.png)
![](map2.png)

# 多键索引
- [Go语言map的多键索引——多个数值条件可以同时查询](https://note.youdao.com/s/D0OQeLbt)-<font color="red">添加键值对到字典时，实际是将键转化为哈希值进行存储</font>

# sync.Map 并发安全的map
**为什么需要这个？**
map 在并发情况下，只读是线程安全的，同时读写是线程不安全的

使用了两个并发函数不断地对 map 进行读和写而发生了竞态问题，fatal error: concurrent map read and map write

**需要并发读写时，一般的做法是加锁，（锁，尽管是读锁也不要乱用   锁冲突），但这样性能并不高**
![](map3.png)
- sync.Map 和 map 不同，不是以语言原生形态提供，而是在 sync 包下的特殊结构
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

    // 将键值对保存到sync.Map
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

# 列表
- 列表是一种非连续的存储容器，由多个节点组成，节点通过一些变量记录彼此之间的关系，列表有多种实现方法，如单链表、双链表等。
- 列表使用<font color="red">container/list包</font>来实现，**内部的实现原理是双链表**

## 初始化
- 变量名 := list.New() 和 var 变量名 list.List
列表与切片和 map 不同的是，列表并没有具体元素类型的限制，因此，列表的元素可以是<font color="red">任意类型</font>，这既带来了便利，也引来一些问题，例如给列表中放入了一个 interface{} 类型的值，取出值后，如果要将 interface{} 转换为其他类型将会发生宕机。
## 插入元素
从前方PushFront 和 从后方PushBack，插入元素，都会返回一个 <font color="red">*list.Element 结构</font>，如果在以后的使用中需要**删除插入的元素**，则只能通过 *list.Element 配合 Remove() 方法进行删除，这种方法可以让删除更加效率化，同时也是双链表特性之一。

InsertAfter(xxx, element) 和 InsertBefore(xxx, element)

## 删除元素
 *list.Element 结构，这个结构**记录着列表元素的值以及与其他节点之间的关系等信息**，从列表中删除元素时，需要用到这个结构进行快速删除。配合 Remove() 方法
## 遍历列表
遍历双链表需要配合 Front() 函数获取头元素，遍历时只要元素不为空就可以继续进行，每一次遍历都会调用元素的 Next() 函数
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