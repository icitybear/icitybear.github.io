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

- Go 是强类型语言，变量类型一旦确定，就不能将其他类型的值赋值给该变量。
- <font color="red">只支持显示转化，然后返回新值。</font>只能显示转封装函数，返回新值, go不支持隐式转比如 func itob(i int) bool { return i != 0 } int转bool
- 强转的基础也是底层要一样，比如都是整形 int=>uint8,<font color="red">强转时如果范围大于后者会被截取截取</font>

# 数据类型转换
- 由于Go语言不存在隐式类型转换，因此所有的类型转换都必须显式的声明
- <font color="red">类型转换精度问题</font>，从一个取值范围较小的类型转换到一个取值范围较大的类型（将 int16 转换为 int32）。当从一个取值范围较大的类型转换到取值范围较小的类型时（将 int32 转换为 int16 或将 float32 转换为 int），会发生精度丢失（截断）的情况。
- 只有<font color="red">相同底层类型的变量之间可以进行相互转换</font>（如将 int16 类型转换成 int32 类型），不同底层类型的变量相互转换时会引发编译错误（如将 bool 类型转换为 int 类型），这种的要<font color="red">通过函数通过输入输出转换（自定义函数强转）</font>
  - 布尔类型不能接受其他类型的赋值，也不支持自动或强制的类型转换。
- 无类型常量， math.Pi 是 math 包的常量，<font color="red">默认没有类型，会在引用到的地方自动根据实际类型进行推导</font>, 范围取最大比如 uint8 < int32 < int等

# 整型之间的转化
- 有符号与无符号以及高位数字向低位数字转化时，需要注意数字的溢出和截断。
- <font color="red">浮点型转化为整型时，小数位被丢弃</font>
- 加减乘除时，注意类型，精度丢失问题

``` go
func TestJisuan(t *testing.T) {

	a := 710 / 100 // 与php不一样会 吞掉小数点 只保留int
	t.Log(a) // 7
	b := float32(710) / float32(100) 
	t.Log(b) //7.1
}
```

# 数值和布尔类型转化
自定义函数

# 整型到字符串
- 通过 Unicode 字符集转化为对应的 UTF-8 编码的字符串 <font color="red">string()</font> **将 byte 数组或者 rune 数组转化为字符串**,byte 是 uint8 的别名，rune 是 int32 的别名，所以也可以看做是整型数组和字符串之间的转化。

# 字符串与数值类型
- 不支持将字符串类型强制转化为数值类型，所以使用strconv 包 封装了函数，通过包方法转换各种类型

## strconv包的方法
- func Itoa(i int) string 整型转字符串
- func Atoi(s string) (i int, err error) 字符串转整型
- Parse 系列函数 将字符串转换为指定类型的值 
  - func ParseXXX(str string) (value XXX, err error)  
- Format 系列函数 将给定类型数据格式化为字符串类型
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
q = strconv.QuoteToASCII("Hello, 世界")  // 将字符串转化为 ASCII 编码 默认是支持utf-8
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

# 字符串的读写

类型	|主要用途	|读写性质	|线程安全	|关键特性
-----|------------|----------|----------|----------
bytes.Buffer	|字节序列的读写(字节缓冲区)	|可读可写	|是	|通用缓冲，支持多种IO操作(处理字节的读写、转换、缓存)
strings.Builder	|构建字符串	|只写	|否	|高效构建字符串（避免拷贝）
strings.Reader	|从字符串读取(字符串读取器	)	|只读	|是（只读天生安全）	|提供读取、定位等功能，将字符串视为数据源(包装成文件流)

## bytes.Buffer （字节缓冲区）
-  在 Go 1.10 引入 strings.Builder 之前，bytes.Buffer 是构建字符串的主要选择,内部使用锁（Mutex），适合并发场景
   -  支持读写交替操作
-  用途：<font color="red"> 类型转换 string 和 []byte, 字节流缓存（如网络请求/响应的缓冲）</font>

``` go
func j3() {
  var buf bytes.Buffer
  buf.WriteString("Hello")
  buf.Write([]byte{32, 100}) // 写入字节 对应asic码
  fmt.Println(buf.String())   // 输出: "Hello d"
}

func j4() {
	var bt bytes.Buffer
	s1 := "chihuo"
	s2 := "golang"
	bt.WriteString(s1)
	bt.WriteString("@")
	bt.WriteString(s2)

	s3 := bt.String() //把Buffer缓存里的转成字符串 不增加内存开销
	fmt.Printf("s1 + s2 = %s\n", s3)
}
```

## strings.Builder（字符串构建器）
- 写入数据（WriteString、WriteByte、WriteRune）后，通过 String() 一次性生成结果字符串。现在对于纯字符串构建，通常首选 strings.Builder。
  - <font color="red"> 实现了io.Writer（写入字节），io.StringWriter（写入字符串），但不可读</font>
  - <font color="red"> String() 方法直接返回底层字节的引用（无拷贝），性能极高</font>
  - 内存增长策略更激进（减少分配次数）
  -  非线程安全，需自行处理并发（如加锁）
- 用途：高频字符串拼接（如生成 JSON、HTML 模板）

``` go
func j5() {
	var builder strings.Builder
	s1 := "chihuo"
	s2 := "golang"
	builder.WriteString(s1)
	builder.WriteString("@")
	builder.WriteString(s2)
	s3 := builder.String()
	fmt.Printf("s1 + s2 = %s\n", s3)
}
``` 

## strings.Reader（字符串读取器）
- strings.Reader 是 Go 标准库中一个重要的类型，<font color="red"> 用于将字符串（string）包装成一个可读取的流（io.Reader）</font>
  - 用于从字符串中高效读取数据。它实现了io.Reader, io.ReaderAt, io.Seeker, io.ByteScanner, io.RuneScanner, io.WriterTo等，使得我们可以像操作文件或字节流一样操作一个字符串
  - 只读特性天生支持并发读
- 用途：<font color="red"> 将字符串作为流处理（如解析字符串数据、模拟文件读取）</font>

``` go

func TestIo(t *testing.T) {
	s1 := "chihuo@golang"
	fmt.Printf("s1 is %v\n", s1)
	stream1 := strings.NewReader(s1)
	fmt.Printf("stream1 is %v\n", stream1) // &strings.Reader{s:"chihuo@golang", i:0, prevRune:-1}
}

func main() {
	r := strings.NewReader("Go语言编程")
	buf := make([]byte, 5)
	
	n, _ := r.Read(buf)
	fmt.Printf("Read %d bytes: %q\n", n, buf[:n]) // Read 5 bytes: "Go语"
	
	// 跳转到开头
	r.Seek(0, 0)
	
	// 逐个字符读取
	for {
		if ch, sz, err := r.ReadRune(); err == nil {
			fmt.Printf("%c (size=%d) ", ch, sz)
		} else {
			break
		}
	}
	// 输出: G (size=1) o (size=1) 语 (size=3) 言 (size=3) 编 (size=3) 程 (size=3)
}

func writeToFile(filename, content string) error {
	r := strings.NewReader(content)
	file, err := os.Create(filename)
	if err != nil {
		return err
	}
	defer file.Close()
	
	_, err = r.WriteTo(file) // 零拷贝高效写入
	return err
}

```


# protobuf3中的float double
- float相当于go的float32, double相当于float64 类型影响数据范围
- <font color="red">所以建议proto3的浮点数类型，定义返回时可以用字符串string, 或者分清数据范围float和double</font>

- gorm的相关modle模型 定义float64, 能正常接收，<font color="red">但是proto3 的float类型（对应是float32），高精度转低精度(精度丢失问题)</font>, 四舍五入问题 float32强转的精度问题，
  - 比如float32导致901848.38 变为901848.4

<font color="red">解决方案：proto3 使用double类型。 或者使用字符串返回浮点数</font>

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
// float32() 强转的时候大转小精度就丢失了
log.Context(ctx).Debugf("db:%+v, to32:%+v, to64:%+v, str:%s", item.MediaCost, float32(item.MediaCost), float64(item.MediaCost), mediaCostStr)
// "message":"db:901848.38, to32:901848.4, to64:901848.38 str:901848.38
```

# <font color="red">kratos框架返回int64字段时，kratos默认会将整型字段输出为字符串。(json序列化的问题)</font>
 - <font color="red">这是因为在HTTP传输中，整型数据会被序列化为JSON格式，而JSON中只支持字符串、数字、布尔值、数组和对象等数据类型。所以为了保持数据的类型完整性，Kratos会将整型字段以字符串形式输出, 这是为了避免在前端处理过程中丢失精度。</font> 如果需要将整型字段作为数字类型输出，可以通过类型转换等方式进行处理。

1. 发起请求的参数数字，json序列化成json数字（在JSON中，数字是以字符串的形式表示的）。
2. encoding/json解码器将JSON数据解码为Go语言中的数据结构。
   - **<font color="red">当解码器遇到一个JSON数字时，它会将其解码为Go语言中的float64类型。</font>**
   - 当数字超过 2^53（约 9e15）时，float64 会丢失精度，导致大整数解析错误,<font color="red">特别是对于int64类型的字段，如果数值很大，转换为float64可能会造成精度损失。</font>
3. 指定解码器解析json字符串里的int64数字, 超出范围精度丢失，使用UseNumber解码器
   - UseNumber方法会使Decoder在解析数字时，**将数字作为json.Number类型（本质是字符串）保存**，而不是直接解析为float64,后续步骤中根据需要将json.Number转换为int64、float64等, 配合结构体定义好字段int64可以直接使用，精度也不会丢失

``` go
// json字符串 解析int64 float64精度丢失
type Data struct {
	ID    string      `json:"id"`
	Value json.Number `json:"value"` // 使用 json.Number 接收
}

type Data2 struct {
	ID    string `json:"id"`
	Value int64  `json:"value"` // 使用 json.Number 接收
}

func TestDecoder3(t *testing.T) {
	// 示例 JSON（包含大整数）
	const jsonStr = `{"id": "test","value": 7044144249855934983}` // int64 最大值
	// 不开启的情况下
	var originData2 Data2
	_ = json.Unmarshal([]byte(jsonStr), &originData2)
	// json_fmt.Data2{ID:"test", Value:7044144249855934983} int64
	fmt.Printf("%+#v %T\n", originData2, originData2.Value)
	// 使用map接收就会异常
	var rawMap map[string]interface{}
	// 使用的是Decode
	if err := json.Unmarshal([]byte(jsonStr), &rawMap); err != nil {
		log.Fatal(err)
	}
	// tag: 此时这里的float64已经出现精度丢失了
	// map[string]interface {}{"id":"test", "value":7.044144249855935e+18} float64
	fmt.Printf("%+#v %T\n", rawMap, rawMap["value"])


	// 创建 Decoder 并启用 UseNumber
	decoder := json.NewDecoder(strings.NewReader(jsonStr))
	decoder.UseNumber() // 关键步骤 启用数字原始解析

	var data0 Data
	err := json.Unmarshal([]byte(jsonStr), &data0)
	if err != nil {
		log.Fatal("Unmarshal解析失败: ", err)
	}
	intValue0, err := data0.Value.Int64()
	if err != nil {
		log.Fatal("转换失败: ", err)
	}
	// json_fmt.Data{ID:"test", Value:"7044144249855934983"} json.Number 7044144249855934983
	fmt.Printf("%+#v %T %d\n", data0, data0.Value, intValue0)

	var data Data // 配合解析器 与不配合一样 只要开启了 数字原始解析
	if err := decoder.Decode(&data); err != nil {
		log.Fatal("Decode解析失败: ", err)
	}

	// 将 json.Number 转为 int64
	intValue, err := data.Value.Int64()
	if err != nil {
		log.Fatal("转换失败: ", err)
	}

	// json_fmt.Data{ID:"test", Value:"7044144249855934983"} json.Number 7044144249855934983
	fmt.Printf("%+#v %T %d\n", data, data.Value, intValue)

	const jsonStr2 = `{"id": "test","value": 7044144249855934983}` // int64 最大值.
	// 创建 Decoder 并启用 UseNumber
	decoder2 := json.NewDecoder(strings.NewReader(jsonStr2))
	decoder2.UseNumber()
	var data2 Data2 // tag: 此时的结构体字段直接定义int64类型
	if err := decoder2.Decode(&data2); err != nil {
		log.Fatal("解析失败: ", err) // 解析失败: EOF
	}
	// json_fmt.Data2{ID:"test", Value:7044144249855934983} int64
	fmt.Printf("%+#v %T", data2, data2.Value)
}

// 解析到map
func TestDecoder4(t *testing.T) {
	jsonStr := `{"value": 9223372036854775807}`

	decoder := json.NewDecoder(strings.NewReader(jsonStr))
	decoder.UseNumber() // 关键：启用数字原始解析

	var raw map[string]interface{}
	// 使用的是Decode
	if err := decoder.Decode(&raw); err != nil {
		log.Fatal(err)
	}

	// 此时 value 是 json.Number 类型
	num, ok := raw["value"].(json.Number)
	if !ok {
		log.Fatal("类型断言失败")
	}

	intValue, err := num.Int64() // 转换为 int64
	if err != nil {
		t.Fatal(err)
	}
	fmt.Println(intValue) // 9223372036854775807
}

// 解析到结构体
func TestDecoder5(t *testing.T) {
	jsonStr := `{"value": 9223372036854775807}`

	var data Data
	if err := json.Unmarshal([]byte(jsonStr), &data); err != nil {
		log.Fatal(err)
	}

	// 安全转换为 int64
	intValue, err := data.Value.Int64()
	if err != nil {
		log.Fatal("转换失败:", err)
	}
	fmt.Println("值:", intValue) // 9223372036854775807
}
```

![alt text](image1.png)

# 浮点数计算精度问题
float32和float64类型的浮点数在进行数学运算时可能会遇到精度问题。这是由于浮点数在计算机中的表示方式决定的，它们无法精确表示所有的小数。当进行浮点数运算时（如乘法），这种精度误差可能会累积，导致结果不如预期那样精确。

## 使用math/big包
- 用固定精度的数学库, math/big包。这些库提供了可以表示任意精度数值的类型，并可以用来执行精确的数学运算。
``` go
import (
    "fmt"
    "math/big"
)

func main() {
    // 使用big.Float表示两个浮点数
    a := new(big.Float).SetFloat64(0.1)
    b := new(big.Float).SetFloat64(0.2)

    // 进行精确的乘法运算
    result := new(big.Float).Mul(a, b)

    // 打印结果
    fmt.Println(result) // 输出: 0.02

    // 设置字符串转float64
    f := new(big.Float).SetString("0.33333333333333333333333333333333333333") // 一个很长的小数
    // 将big.Float转换为float64
    float64Value, _ := f.Float64()
    fmt.Println(float64Value) // 输出: 0.3333333333333333
}

// big.Float提供了Int方法，可以将其转换为big.Int类型，然后再转换为普通的整数类型
func main() {
  // 创建一个big.Float并给它赋值
  f := new(big.Float).SetString("123.456")

  // 将big.Float转换为big.Int
  i := new(big.Int)
  f.Int(i) // 转换  使用Int方法会丢失小数部分，因为它只获取big.Float的整数部分。

  // 输出big.Int的值
  fmt.Println(i) // 输出: 123

  // 将big.Int转换为int64
  int64Value := i.Int64()  // 输出: 123
}

// Format 方法可以用来将浮点数格式化为字符串。类似于 fmt.Sprintf，但是它是为 big.Float 类型量身定制的。该方法接受两个参数：格式（和 fmt 包中的格式化谓词类似）和精度。
func main() {
    // 创建并设置一个big.Float值
    f := new(big.Float).SetPrec(128).SetFloat64(123.456)

    // 格式化big.Float为字符串
    // 'f' 表示浮点数记数法。第二个参数表示小数点后的精度。
    // 此处设置为10，表示转换后的字符串小数点后有10位数字。
    s := f.Text('f', 10)

    fmt.Println(s) // 输出: 123.4560000000
}

// 
  money := new(big.Float).SetFloat64(123.456)
  bs := new(big.Float).SetFloat64(100)
  valbig := new(big.Float).Mul(money, bs)
  // int转float Text会四舍五入0位小数
  moneyFloat, _ := strconv.ParseFloat(valbig.Text('f', 0), 64)
```

## 字符串与decimal数值
- github.com/shopspring/decimal
``` go
// 字符串与数值转字符串
func TestAoti(t *testing.T) {
	str := "7431820065157906458"
	// string转int     Itoa
	s, _ := strconv.Atoi(str) // 整形最大范围 9223372036854775807
	fmt.Println(s)            // 7431820065157906458

	// 指数形式表示的 浮点型float64 转字符串
	floatNum := 7.431820065157906e+18
	strNum := strconv.FormatFloat(floatNum, 'f', 15, 64)  // 'f' 表示没有指数部分，保留15位小数 一般会是2保留2位
	fmt.Println(strNum)                                   // 7431820065157906432.000000000000000
	strNum1 := strconv.FormatFloat(floatNum, 'f', -1, 64) // 'f' 表示去掉指数部分 -1没有小数位
	fmt.Println(strNum1)                               // 7431820065157906000

	// int64转string
	// strconv.FormatInt(int64(num1), 10) // 10进制

	// string转float64 float32
	num, _ := strconv.ParseFloat(fmt.Sprintf("%.8f", floatNum), 64) // 参数只有32或者64
	fmt.Println(num)                                                // 输出原始的 float64 数值 7.431820065157906e+18

	// 使用 decimal.NewFromFloat 创建一个 decimal.Decimal 实例
	decimalValue := decimal.NewFromFloat(floatNum)
	// 乘以 100，使用 decimal 的 Mul 方法
	decimalValue = decimalValue.Mul(decimal.NewFromInt(100))

	// 将结果转换回 float64
	res, _ := decimalValue.Float64()
	fmt.Println(res) // 7.431820065157906e+20 因为多了2位

	// string转decimal 7431820065157906000"
	d, _ := decimal.NewFromString(strNum1)
	fmt.Println(d.String()) // 转string "7431820065157906000"
	res2, _ := d.Float64()  // 转float64 7431820065157906000
	fmt.Println(strconv.FormatFloat(res2, 'f', -1, 64))

}

// 通用转换方式  int也适用
func GetInterFaceDecimal(v interface{}) decimal.Decimal {
	var r decimal.Decimal
	var err error
	switch v.(type) {
	case uint:
		r = decimal.NewFromInt(int64(v.(uint)))
		break
	case int8:
		r = decimal.NewFromInt(int64(v.(int8)))
		break
	case uint8:
		r = decimal.NewFromInt(int64(v.(uint8)))
		break
	case int16:
		r = decimal.NewFromInt(int64(v.(int16)))
		break
	case uint16:
		r = decimal.NewFromInt(int64(v.(uint16)))
		break
	case int32:
		r = decimal.NewFromInt32(v.(int32))
		break
	case uint32:
		r = decimal.NewFromInt(int64(v.(uint32)))
		break
	case int64:
		r = decimal.NewFromInt(v.(int64))
		break
	case uint64:
		r = decimal.NewFromInt(int64(v.(uint64)))
		break
	case float32:
		r = decimal.NewFromFloat32(v.(float32))
		break
	case float64:
		r = decimal.NewFromFloat(v.(float64))
		break
	case string:
		r, err = decimal.NewFromString(v.(string))
		if err != nil {
			return decimal.NewFromInt(0) // 报错的的情况下默认0
		}
		break
	case int:
		r = decimal.NewFromInt(int64(v.(int)))
		break
	case nil:
		return decimal.NewFromInt(0)
	case decimal.Decimal:
		r = v.(decimal.Decimal)
	default:
		return decimal.NewFromInt(0)
	}
	return r
}

func DivideFromInterface(x interface{}, y interface{}) decimal.Decimal {
	xd := GetInterFaceDecimal(x)
	yd := GetInterFaceDecimal(y)
	rs := xd.Div(yd)
	return rs
}

func MulFromInterface(x interface{}, y interface{}) decimal.Decimal {
	xd := GetInterFaceDecimal(x)
	yd := GetInterFaceDecimal(y)
	rs := xd.Mul(yd)
	return rs
}

// string转千分计数且考虑小数点
func ThousandSeparate(str string) string {
	arr := strings.Split(str, ".")
	integerPart := arr[0]
	var result strings.Builder

	// 处理整数部分
	for i, ch := range integerPart {
		// 从右往左每三位加逗号（当剩余位数是3的倍数且不在起始位置时）
		if i > 0 && (len(integerPart)-i)%3 == 0 {
			result.WriteByte(',') // 单字节
		}
		result.WriteRune(ch) // 4个字节对应的Unicode码点
	}

	// 处理小数部分
	if len(arr) > 1 {
		result.WriteByte('.')
		result.WriteString(strings.Join(arr[1:], "."))
	}

	return result.String()
}
```
