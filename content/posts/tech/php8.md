---
title: "php函数" #标题
date: 2023-07-01T21:47:48+08:00 #创建时间
lastmod: 2023-07-01T21:47:48+08:00 #更新时间
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
    image: "" #图片路径：posts/tech/文章1/picture.png
    caption: "" #图片底部描述
    alt: ""
    relative: false

# reward: true # 打赏
mermaid: true #自己加的是否开启mermaid
---

# 自定义函数
- function
- 形参与实参，值传递和引用传递
- 从 PHP 7 开始，支持对传入参数和返回值声明数据类型 参数类型声明 ?int, int，返回值声明: int
- 参数类型自动转换
- 可变数量的参数列表,代表参数列表的 $parameter 前面有一个 ... 前缀
- 默认参数,需要放到参数列表最后

# 函数变量
函数在 PHP 中也可以看作是一等公民（first class），可以赋值给变量进行调用

# 可变函数
与可变变量一样的道理，由于 $a 是一个函数类型变量，并且 PHP 是动态类型语言，所以我们还可以像操作基本类型变量那样**将其他函数类型值**赋值给 $a，这些函数类型值包括匿名函数和非匿名函数

# 内置函数
- 字符串函数, 数组函数, 文件函数, 数学函数等

# 匿名函数（闭包函数）
没有函数名
闭包，理解成“定义在一个函数内部的函数“。在本质上，闭包是将函数内部和函数外部连接起来的桥梁。 <font color="red">在php中,闭包函数一般就是匿名函数.</font> 
## 作用域
- 支持在函数体中直接引用上下文变量(继承父作用域的变量),其他引用该文件的代码就可以间接引用当前父作用域下的变量，如果是在类方法中定义的匿名函数，则可以直接引用相应类实例的属性,use是连接闭包和外界变量
- 基于 global 关键字通过全局变量引用函数体外部定义的变量
- 从父作用域中继承变量与使用全局变量是不同的，全局变量存在于一个全局的范围，无论当前在执行的是哪个函数，<font color="red">而闭包的父作用域是定义该闭包的函数</font>，不一定是调用它的函数。


``` php
<?php
// 正常匿名函数
$fun = function($name){
    printf("Hello %s\r\n",$name);
};
echo $fun('Tioncico'); // 函数变量直接调用

// 匿名函数当函数的参数 回调
function funcTest($callback){
    return $callback();
}
$str1 = "hello,";
$str2 = "Tioncico,";
funcTest(function ()use($str1,$str2){
    echo $str1,$str2."\n";
    return 1; // 因为a的形参（函数变量） 回调函数有返回值
});
```
匿名函数不能调用所在代码块的上下文变量，而需要通过使用use关键字。
通过use继承父作用域变量包

# 闭包作用
1. 减少foreach的循环的代码
``` php
<?php
// 一个基本的购物车，包括一些已经添加的商品和每种商品的数量。
// 其中有一个方法用来计算购物车中所有商品的总价格。该方法使用了一个closure作为回调函数
class Cart
{
    const PRICE_BUTTER = 1.00;
    const PRICE_MILK = 3.00;
    const PRICE_EGGS = 6.95;
    
    protected $products =array();
    
    public function add($product,$quantity)
    {
        $this->products[$product] = $quantity;
    }
    
    public function getQuantity($product)
    {
        return isset($this->products[$product]) ? $this->products[$product] :
        FALSE;
    }
    
    // $tax 税率
    public function getTotal($tax)
    {
        $total = 0.00;
        $callback = function ($quantity, $product)use ($tax, &$total){
            $pricePerItem = constant(__CLASS__ ."::PRICE_" .strtoupper($product));
            $total += ($pricePerItem *$quantity) * ($tax + 1.0);
        };
        //  $total要改变值的话 要进行取址&引用在函数中，数组的键名和键值是参数2,1。也可以自定义传数据数组进入
        // 使用 匿名函数作为回调函数 （闭包）
        array_walk($this->products, $callback);
        return round($total, 2);
    }

}

$my_cart =new Cart;

// 往购物车里添加条目
$my_cart->add('butter', 1);
$my_cart->add('milk', 3);
$my_cart->add('eggs', 6);

// 打出出总价格，其中有 5% 的销售税.
print $my_cart->getTotal(0.05) . "\n";
// The result is 54.29
```
2. 减少函数的参数
3. 用于递归
``` php
<?php
$fib = function($n)use(&$fib) {
    if($n == 0 || $n == 1) return 1;
    return $fib($n - 1) + $fib($n - 2);
};

echo $fib(2) . "\n";// 2  

echo $fib(5); // 8
```
use使用了&，这里不使用&会出现错误fib(fib(n-1)是找不到function的（前面没有定义fib的类型）$fib为要进行递归的函数
4. 延迟绑定
``` php
<?php
$result = 0;
$one =function() { var_dump($result); };
$two =function()use ($result) { var_dump($result); };
$three =function()use (&$result) { var_dump($result); };

$result++;

$one(); // NULL

$two(); // int(0)

$three(); // int(1)
```
<font color="red">**使用引用和不使用引用就代表了是调用时赋值，还是申明时候赋值**</font>
