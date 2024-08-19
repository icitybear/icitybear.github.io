---
title: "php基础" #标题
date: 2023-07-01T11:01:24+08:00 #创建时间
lastmod: 2023-07-01T11:01:24+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- php
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
    image: "posts/tech/php7/php.png" #图片路径：posts/tech/文章1/picture.png
    caption: "" #图片底部描述
    alt: ""
    relative: false
    hidden: true
# reward: true # 打赏
mermaid: true #自己加的是否开启mermaid
---

# php与html
- 在 Web 2.0 时代，PHP 进一步进化为 PHP:Hypertext Preprocessor，即超文本处理器，而 HTML 则是 HyperText Markup Language 的缩写，也就是超文本标记语言
- PHP 脚本在 HTML 文档中只是一种特殊标记而已，并且可以在 HTML 文档中直接编写任何 PHP 脚本代码，**然后将文档保存为 .php 文件，就可以被 PHP 解释器解析和执行。**
- 在 PHP 文件中，既可以编写纯 PHP 代码，也可以混合 HTML + PHP 代码进行编程（在 HTML 中嵌入 PHP 代码需要通过完整的 <?php 和 ?> 进行包裹）。在混合 HTML 的 PHP 文件中，还可以引入 CSS、JavaScript 代码让渲染效果和页面功能更加丰富，这些在 PHP 中都是原生支持的，不需要引入任何额外的设置、扩展包，并且 PHP 本身是动态解释型语言，代码修改无需编译即可立即生效，所以在 Web 开发中非常高效。

# php内置web服务器
[内置web服务器](https://baijiahao.baidu.com/s?id=1721302936763859973&wfr=spider&for=pc)

```
php -S localhost:2333 [-t html] [-c php.ini] [router.php]
```

# 变量
1. PHP 是弱类型语言，变量类型在运行时确定，所以不需要声明数据类型
2. PHP 变量的声明和初始化是一步完成的，不需要也不支持单独的声明语句
3. PHP 变量名都以 $ 作为前缀，所以支持将系统关键字和保留字作为变量名,$ 之后具体的变量名不能以数字开头,只支持字母（支持中文字符，不过我们尽量使用 ASCII 字符，以免出现意想不到的问题）、数字、下划线
4. PHP 变量名大小写敏感，变量名一致，大小写不一致，会被看作不同的变量
   
## 可变变量
变量名前再加上一个 $ 前缀，将对应变量值作为一个变量名进行引用。

``` php
<?php 

$greeting = "你好，PHP！";
$varName = "greeting";
echo $$varName;

你好，PHP！
```
因为 $varName 的变量值是 greeting，所以当我们调用 $$varName 时，$varName 被替换成 greeting，因此实际上引用的是 $greeting，由于 $varName 的值可以动态设置，所以也就可以实现了一个可变变量。

# 常量

- 常量存在的意义就是设置运行期「只读变量」，保护「这些变量」运行期间不被更改。
- 常量名不需要 $ 前缀（也不能设置），并且为了和变量做区分，通常都是以大写字母进行命名（同样大小写敏感）,其他规则与变量命名一样
  
1. 通过 define 函数 定义的常量全局有效，所以通常在项目初始化期间通过这种方式定义全局常量。

2. 通过const 修饰符的方式定义常量，用于在类中设置只读属性（类常量）,但是也能用在全局

``` php
<?php 
define("LANGUAGE", "PHP");
define("AUTHOR", "陈xx");

const FRAMEWORK = "xy"; // 一般用于类常量
```
# 基本数据类型
**var_dmup() 函数 打印变量**
## 字符串 string
is_string
1. 单引号字符串中引用变量不会对变量值进行解析，如果是双引号，则会对引用变量值进行解析
2. 单引号和双引号中都可以使用转义字符\，但是，在单引号定义的字符串中只能转义单引号\' 和转义符(反斜线本身)本身\\ 转义符本身
3. 双引号定义的字符串中，PHP 可以转义更多的特殊字符。

- [常见字符串函数](https://note.youdao.com/s/6uleC7H8)
- [php字符串操作-url加密,base64,实体 防SQL注入 防xss](https://note.youdao.com/s/4fwrrAZs)
- [json报错问题](https://note.youdao.com/s/1AHjDOZl)
- [编码问题（导致乱码），emoji](https://note.youdao.com/s/Baa5sAAZ)
- [字符串转unicode编码（全转）](https://note.youdao.com/s/Ob1jjwUi)
- [通用过滤方法](https://note.youdao.com/s/JLsQFUdg)
  
## 整型 int/integer
is_int/is_integer
- PHP 中，整型类型没有位数之分，所有的整型都统归 int/integer 类型，并且不支持无符号整型。
- PHP_INT_MIN和 PHP_INT_MAX 这两个内置

## 浮点型 float（单精度）和 double（双精度）
is_float/is_double
- 因为对于确定的十进制小数而言，使用二进制永远无法精确表示，所以不能直接对浮点型进行相等比较，因为即使字面上（十进制）相等，实际底层处理后的二进制数据并不相等

## 布尔类型 bool
is_bool
- 在PHP中，无法使用(bool)类型转换或者settype()函数把字符串的"true"和"false"转成布尔型。如果使用上述两种办法，会始终返回true。
``` php
<?php 
function is_true($val, $return_null=false){
    $boolval = ( is_string($val) ? filter_var($val, FILTER_VALIDATE_BOOLEAN, FILTER_NULL_ON_FAILURE) : (bool) $val );
    return ( $boolval===null && !$return_null ? false : $boolval );
}
```

## 基本数据类型之间的转化
- 只需要在变量名前通过添加 (目标转化类型) 强制转化即可
- 会隐式转换
``` php
$array = [10,'11-']
var_dump(in_array(11, $array)); 
var_dump(11 == '11-'); 
var_dump('11' == '11-');

bool(true)
bool(true)
bool(false)
```
![alt text](image.png)

## 扩展bcmath精度问题
-  bc 函数要求2个string，当传递 number 时，php 会自动将 number 转成对应的 string
-  <font color="red">字符串转数值, 科学计数法值作为参数的坑</font>
``` php
$nums = [75.99999, 20, 3, 1, 0.00001];
$total = 0;
foreach ($nums as $num) {
    $total = bcadd($total, $num, 5);
}
echo $total . PHP_EOL;
// 输出了99.99999，为啥0.00001没计算，明显也是保留5位小数点
// 原因是 bc 函数要求2个string，当传递 number 时，php 会自动将 number 转成对应的 string
// 最后的 0.00001 预期转换为 '0.00001' 但实际被转换成 '1.0E-5' 导致最后的 0.00001 不被计算到 $total 里（bcadd执行错误）

$nums = [75.99999, 20, 3, 1, 0.00001];
$total = '0';
foreach ($nums as $num) {
    // (string) $num 或 strval($num) 都会转成 '1.0E-5'
    $total = bcadd($total, sprintf('%.5f', $num), 5);
}
echo $total . PHP_EOL; // output '100.00000'
```
![alt text](image1.png)

## 过滤器 函数 filter_xxx()
[filter过滤函数](https://note.youdao.com/s/6iPZ7Idw)

# 数组
**通过 print_r() 函数打印数组**
在静态语言（C、Java、Go）中，数组的定义通常是同一类型数据的连续序列，PHP 的数组从功能角度来说更加强大，可以包含任何数据类型，支持无限扩容，并且将传统数组和字典类型合二为一，在 PHP 中，<font color="red">**传统的数组对应的是索引数组，字典类型对应的是关联数组**</font>，这得益于 PHP 底层通过<font color="red">哈希表</font>实现数组功能

## 索引数组
- 索引数组指的是数组的键为隐式数字，并且会自动维护，就像静态语言的数组一样
- PHP 索引数组的索引值和其他语言一样，都是从 0 开始。
- PHP 数组支持任意类型数据
- 增删改查与索引位置,count与unset
- PHP 数组底层是哈希表驱动，所以支持无限扩容
<font color="red">PHP 索引数组就要比传统静态语言的数组灵活的多，因为摆脱了数据类型和初始大小这两把枷锁。</font>\

## 关联数组 (字典)
- 显式指定数组元素的键,可以部分指定
- 增删改查
----
- [常见数组函数](https://note.youdao.com/s/Uc6YcGdf)
- [排序](https://note.youdao.com/s/5Tu8RW5Y)
- [二维数组排序](https://note.youdao.com/s/GPdhibUK)

# 运算符
**[官方文档](https://www.php.net/manual/zh/language.operators.php)**
- 算术运算符
自增/自减运算符位于*变量之前*时，运算之后**直接返回变更后的值**，而自增/自减运算符位于*变量之后*时，当前操作返回值还是原始值，直到下次被调用，才会引用更新后的变量值.
- 比较运算符
![](compare.jpg)
- 逻辑运算符
- 其他运算符
赋值运算符
位运算符
错误控制运算符
执行运算符
字符串运算符
数组运算符
类型运算符

- 运算符优先级