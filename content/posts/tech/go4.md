---
title: "3.0-go的复合类型-数组和切片（nil空值）" #标题
date: 2023-07-13T23:21:14+08:00 #创建时间
lastmod: 2023-07-13T23:21:14+08:00 #更新时间
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

# 数组 [] 
- **<font color="red">数组是固定长度的、同一类型的数据集合, 值类型</font>, 与切片显著区别：长度编译时就确定不变**
- 数组元素通过 {} 包裹，然后通过逗号分隔多个元素
- [...]语法糖省略数组长度的声明,会在编译期自动计算出数组长度。
- 初始化的时候，如果没有填满，则空位会通过对应的元素类型零值填充,初始化指定下标位置的元素值
- 与变量一样，函数内通过 := 进行<font color="red">一次性声明和初始化</font>
- 数组长度在声明后就不可更改(也就不能动态扩容), 长度编译时就能获取，数组的长度是该数组类型的一个内置常量，可以用 Go 语言的内置函数 len() 来获取
- 比较时 数组包括长度和值类型

``` go
// 可以先声明再赋值
var a [8]byte // 长度为8的数组，每个元素为一个字节
var b [3][3]int // 二维数组（9宫格）
var c [3][3][3]float64 // 三维数组（立体的9宫格）
var d = [3]int{1, 2, 3}  // 声明时初始化
var e = new([3]string)   // 通过 new 初始化

b := [...]int{1, 2} // 与变量一样，函数内通过 := 进行一次性声明和初始化
arr2 := [4]int{1: 34, 2: 5} // 指定下标位置
```

# 数组访问
- <font color="red">使用数组下标访问</font> 超出这个范围编译时会报索引越界异常 invalid array index 5 (out of bounds for 5-element array)
# 遍历 
- for len
- 遍历 range range 表达式返回两个值，第一个是数组下标索引值，第二个是索引对应数组元素值，可用匿名变量_
- 通过range获取数组的值 -> 不能修改原数组中结构体的值： <font color="red">只是值副本 只能通过下标</font>

## 多维数组 
- 每个元素可能是个数组，在进行循环遍历的时候需要多层嵌套循环

## <font color="red">数组缺点</font>
- 不能动态添加元素到数组
- 值类型，作为参数传递到函数时，<font color="red">传递的是数组的值拷贝</font>，也就是说，会先将数组拷贝给形参，然后在<font color="red">函数体中引用的是形参而不是原来的数组</font>，当我们在函数中对数组元素进行修改时，并不会影响原来的数组。<font color="red">当数组很大时，值拷贝会降低程序性能</font>

**<font color="red">为什么需要切片</font>**
- 需要一个引用类型的、支持动态添加元素的新「数组」类型，切片类型

# 切片slice

``` go
// 源码结构体
type slice struct {
    array unsafe.Pointer //指向存放数据的数组指针
    len   int            //长度有多大
    cap   int            //容量有多大
}
```
- 由三个部分构成 —— <font color="red">指针、长度和容量</font>
- 切片的类型字面量中只有元素的类型，没有长度： []不定长
- 基于数组，做了一层封装
- **可变长度的、同一类型元素集合**，<font color="red">切片的长度可以随着元素数量的增长而增长（但不会随着元素数量的减少而减少）</font>
- 创建 基于数组、切片和make直接创建，<font color="red">本质都是基于数组</font>
- <font color="red">切片则可以看作是数组某个连续片段的引用</font>
![](slice1.png)

## 基于数组
![](slice2.png)
- array[start:end] 左闭右开的集合, 支持缺省写法
- 切片可以只使用数组的一部分元素或者整个数组来创建
- <font color="red">切片则可以看作是数组某个连续片段的引用</font>

## 基于切片
- 基于切片，本质也是基于数组
- slice[start:end]
``` go
func TestSliceShareMemory(t *testing.T) {
	//下标是0开始计数
	year := []string{"Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep",
		"Oct", "Nov", "Dec"}
	t.Log(year, len(year), cap(year))
	//slice[] （定位到开始位置，然后切对应大小）
	//不改变原来的内存地址 切片包含 开始位置 长度 容量大小
	Q3 := year[8:12] //超出长度会编译报错 不包括结尾位置
	t.Log(Q3, len(Q3), cap(Q3))
	Q4 := year[:0] //缺省写法 [] 0 12
	t.Log(Q4, len(Q4), cap(Q4))
	//year = append(year, "end")
	// 容量问题
	Q2 := year[3:6] //不包括结尾位置
	t.Log(Q2, len(Q2), cap(Q2))
	summer := year[5:8]
	t.Log(summer, len(summer), cap(summer))
	//更改其中一个的值后 year也跟着变了
	summer[0] = "Unknow" // year[5] Q2[2]
	t.Log(Q2)
	t.Log(year)
}
```

## 直接创建 make
![](slice3.png)
内置函数 make( []Type, size, cap ) 可以用于灵活地创建切片。
- 默认填充对应类型的零值
- Type 是指切片的元素类型，size 指的是为这个类型分配多少个元素，cap 为预分配的元素数量，这个值设定后不影响 size，只是能提前分配空间，降低多次分配空间造成的性能问题。容量不会影响当前的元素个数
- <font color="red">使用 make() 函数生成的切片一定发生了内存分配操作，与其他2种方式给定开始与结束位置（包括切片复位）的切片（slice [开始位置 : 结束位置]）只是将新的切片结构指向已经分配好的内存区域，设定开始与结束位置，不会发生内存分配操作。</font>
- Go 底层还是会有一个<font color="red">匿名数组</font>被创建出来，然后调用基于数组创建切片的方式返回切片，只是上层不需要关心这个匿名数组的操作而已

## 遍历切片
与遍历数组一致

## 声明切片和空切片 初始化{}
- 用途：接口返回时，比如返回map类型，序列化时时nil （null）, 与[]的区别
- **<font color="red">空切片指声明了不填充默认值的slice{}（v值为[]）, 只声明不赋值是有区别的（v值nil），声明但是分配的是默认零值（v值[0 0]）</font>**
``` go
func TestSliceComparing(t *testing.T) {
	a := []int{1, 2, 3, 4}
	b := []int{1, 2, 3, 4} //切片
	//c := [...]int{1, 2, 3, 4} 数组
	// if a == b { //切片只能和nil比较 内含指针与指针不能比较，计算
	// 	t.Log("equal")
	// }
	t.Log(a, b)
	var c []int        //只声明 不分配内存 nil
	var d []int        //申明
    x := []int{} // 注意等号 var x = []int{}
	d = make([]int, 2) //分配内存 并且会默认填充对应类型的零值
	if c == nil {
		// 只声明一个切片 未分配内存
		t.Log("c equal nil") // 会执行
	}
	if d == nil {
		// 空切片 分配内存了 make会填充默认值
		t.Log("d equal nil") // 不执行
	}
    if x == nil {
		// 空切片 不填充默认值
		t.Log("t equal nil") // 不执行
	}
	t.Logf("%v %T", c, c) //nil [] []int
	t.Logf("%v %T", d, d) //非nil [0 0] []int
    t.Logf("%v %T", x, x) //非nil [] []int
	// 声明一个空切片 注意有等号
	var e = []int{}
	// 声明并初始化, 只是未填充,这里用具体类型int string会报错 nil [] []interface {}
	// var e []interface{} 要使用这个var e = []interface{} Compilation failed
	if e == nil {
		// 空切片分配内存了
		t.Log("e equal nil")
	}
	t.Logf("%v %T", e, e) //无nil [] []int

	// 指针 创建该类型的指针 分配内存 *f 取指针指向变量的值 零值
	var f = new(int)
	if f == nil {
		t.Log("f equal nil")
	}
	t.Logf("%v %T %d", f, f, *f) // 0xc000014328 *int 0

}
```

- 声明但未使用(未赋值)的切片的默认值是 nil  == nil 只声明
- 使用了{}, 本来会在{}中填充切片的初始化元素，<font color="red">这里没有填充(因为也不知道长度)，所以切片是空的</font>，但是此时的 已经被分配了内存，只是还没有元素。因此和 nil 比较时是 false。 空切片（[]）, != nil 无默认值
- 切片是动态结构（引用类型），只能与 nil 判定相等，不能互相判定相等(切片直接不能互相比较)。(切片可以看做是操作数组的指针)
- make空切片（先声明再赋值也一样）,<font color="red">发生了内存分配操作,并且初始化默认值</font> != nil 有默认值, make返回的是引用类型本身

## make和new区别 
{{< innerlink src="posts/tech/go28.md" >}}  

## 扩容append
Go 语言内置的 cap() 函数和 len() 函数来获取某个切片的容量和实际长度
- 对于基于数组和切片创建的切片而言，默认容量是从切片起始索引到对应底层数组的结尾索引
- 对于通过内置 make 函数创建的切片而言，在没有指定容量参数的情况下，默认容量和切片长度一致
- 函数append() 的第二个参数是一个不定参数
- 直接将一个切片追加到另一个切片的末尾 xxx...
- 如果使用了make初始化了容量，那么可以在range里，通过下标赋值效率上肯定远胜 append（就算编译器会做优化，从代码书写上来说，也是下标赋值更直观）

``` go
var oldSlice = make([]int, 5, 10)
newSlice := append(oldSlice, 1, 2, 3)
appendSlice := []int{1, 2, 3, 4, 5}
newSlice := append(oldSlice, appendSlice...)  // 注意末尾的 ... 不能省略
```
### 自动扩容
使用 append() 函数向切片中添加元素。在使用 append() 函数为切片动态添加元素时
- 追加的元素个数超出 oldSlice 的默认容量，则底层会自动进行扩容 (可以看源码)
  - <font color="red">生成一个容量更大的切片</font>，然后把原有的元素和新元素一并拷贝到新切片中。
  - 默认2倍，当原切片的长度大于或等于 1024 时，Go 语言将会以原容量的 1.25 倍作为新容量的基准（后续不一定是1.25）
- **<font color="red">切片自动扩容后，会返回新切片，对应切片的地址也会发生改变</font>, 不扩容就不变**
- 在切片开头添加元素一般都会导致内存的重新分配，而且会导致已有元素全部被复制 1 次，因此，从切片的开头添加元素的性能要比从尾部追加元素的性能差很多
  
### 内容复制 copy
- 内置函数 copy()，用于将元素从一个切片复制到另一个切片。如果两个切片不一样大，就会<font color="red">按其中较小的那个切片的元素个数进行复制。</font>
- 实现删除头3个元素, slice3[:copy(slice3, slice3[3:])] 
## 删除
- 通过切片的切片实现的「伪删除」数据还是那份
- append 函数和 copy 函数实现切片元素的「删除」
- copy 之所以可以用于删除元素，是因为其返回值是拷贝成功的元素个数，我们可以<font color="red">根据这个值完成新切片的设置</font>从而达到「删除」元素的效果,和动态增加元素一样，**<font color="red">原切片的值并没有变动，而是创建出一个新的内存空间来存放新切片并将其赋值给其它变量。</font>**
- 删除返回的切片，**指针是否变化根据起始指针是否变了**

``` go
func TestDel(t *testing.T) {
	// 通过切片的切片
	slice0 := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	slice1 := slice0[:len(slice0)-5] // 删除 slice3 尾部 5 个元素 这种不变 起始指针不变 下标0
	slice2 := slice0[5:]             // 删除 slice3 头部 5 个元素 会变化 起始指针变了 下标5
	// slice2 := append(slice0[:0], slice0[5:]...) // 2种删除头部的 返回的切片地址不一样， 这种不变 起始指针不变 下标0
	fmt.Printf("%p, %p, %p\n", slice0, slice1, slice2) // 地址是头部指针 0x1400011a050, 0x1400011a050, 0x1400011a078

	slice3 := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	fmt.Printf("%p\n", slice3) // 0x1400011a0a0
	// 通过append 删除并没有自动扩容，所以不会返回新切片地址，还是旧切片
	slice4 := append(slice3[:0], slice3[3:]...) // 删除开头三个元素
	fmt.Printf("%p, %p\n", slice3, slice4)      // 0x1400011a0a0, 0x1400011a0a0
	slice5 := append(slice3[:1], slice3[4:]...) // 删除中间三个元素
	fmt.Printf("%p, %p\n", slice3, slice5)      // 0x1400011a0a0, 0x1400011a0a0
	slice6 := append(slice3[:0], slice3[:7]...) // 删除最后三个元素
	fmt.Printf("%p, %p\n", slice3, slice6)      // 0x1400011a0a0, 0x1400011a0a0
	slice7 := slice3[:copy(slice3, slice3[3:])] // 删除开头前三个元素 这种不变 起始指针不变 下标0
	fmt.Printf("%p, %p\n", slice3, slice7)      // 0x1400011a0a0, 0x1400011a0a0
}
```

### 数据共享问题 (重新分配内存)
- **切片结构体，在结构体中使用指针存在不同实例的数据共享问题**

比如：slice2 是基于 slice1 创建的，它们的数组指针指向了同一个数组，因此，修改 slice2 元素会同步到 slice1，因为修改的是同一份内存数据，这就是数据共享问题
- 解决方案 (重新分配内存)
``` go
slice1 := make([]int, 4)
// slice1 := make([]int, 4, 5) //初始化的容量是 5，比长度大，执行append 的时候没有进行扩容，也就不存在重新分配内存操作。
slice2 := slice1[1:3] // slice2 是基于 slice1 创建的
// append 函数会重新分配新的内存，然后将结果赋值给 slice1，
// 这样一来，slice2 会和老的 slice1 共享同一个底层数组内存，不再和新的 slice1 共享内存
slice1 = append(slice1, 0)
slice1[1] = 2
slice2[1] = 6

fmt.Println("slice1:", slice1)
fmt.Println("slice2:", slice2)
```
**一定要重新分配内存空间，如果没有重新分配，依然存在数据共享问题**

{{< innerlink src="posts/tech/php5.md" >}}  
**比如 PHP，类似问题就是引用对象共享，在涉及到引用对象属性的复合对象集合遍历时，很多初学者可能都遇到过这个问题，其实就是浅拷贝导致的不同对象引用了同一个对象属性，要解决这个问题，需要通过深拷贝将对象及嵌套引用的对象重新克隆一份出来，避免内存共享**

# 多维切片
- 每个元素都是一个切片

``` go
//声明一个二维切片 命名slice
var slice [][]int
//为二维切片赋值
slice = [][]int{{10}, {100, 200}}

// 声明一个二维整型切片并赋值
slice := [][]int{{10}, {100, 200}}
```
![](slice4.png)

# nil 空值/零值
- 基本类型的零值：布尔类型的零值（初始值）为 false，数值类型的零值为 0，字符串类型的零值为空字符串""，
- nil 是 map、slice、pointer、channel、func、interface 的零值。预定义好的标识符
- nil 标识符是不能比较的
- nil 没有默认类型  use of untyped nil
- 不同类型 nil 的指针是一样的  %p都是 0x0
- 不同类型的 nil 值 unsafe.Sizeof( m ) 大小可能不同，大小取决于编译器和架构， 也不能比较


``` go
func main() {
    var m map[int]string
    var ptr *int
    var c chan int
    var sl []int
    var f func()
    var i interface{}
    fmt.Printf("%#v\n", m)
    fmt.Printf("%#v\n", ptr)
    fmt.Printf("%#v\n", c)
    fmt.Printf("%#v\n", sl)
    fmt.Printf("%#v\n", f)
    fmt.Printf("%#v\n", i)
}
// 打印结果
map[int]string(nil)
(*int)(nil)
(chan int)(nil)
[]int(nil)
(func())(nil)
<nil>

```

- <font color="red">参数为非基本类型，使用nil作为值时，要使用对应类型的零值（先声明类型）</font>
``` go
func TestArraySection(t *testing.T) {
	arr3 := [...]int{1, 2, 3, 4, 5}
	arr3_sec := arr3[:]

	arr1 := []int{1, 2, 3, 4, 5}
	var arr2 []int
	arr4 := append(arr2, arr1...)
	t.Log(arr4)
	// first argument to append must be a slice; have untyped nil
	// arr5 := append(nil, arr1...) // 但是不能直接使用nil
	// t.Log(arr5)

	t.Log(arr3_sec)
}
```