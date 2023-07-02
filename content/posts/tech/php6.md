---
title: "php-COW机制(写时复制)" #标题
date: 2023-06-28T21:00:33+08:00 #创建时间
lastmod: 2023-06-28T21:00:33+08:00 #更新时间
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

# 简介

写时复制（Copy-on-Write，也缩写为COW），顾名思义，就是在写入时才真正复制一份内存进行修改。 COW最早应用在linux系统中对线程与内存使用的优化，后面广泛的被使用在各种编程语言中，如C++的STL等。 在PHP内核中，<font color="red">COW也是主要的内存优化手段。</font> 在前面关于变量和内存的讨论中，引用计数对变量的销毁与回收中起着至关重要的标识作用。 **引用计数**存在的意义，就是为了使得COW可以正常运作，从而实现对内存的优化使用


# 详细作用
``` php
<?php
var_dump(memory_get_usage());//先打印出当前内存情况
$arr = array_fill(0, 100000, 'tioncico');//生成一个0-100000键的数组
var_dump(memory_get_usage());//打印内存
$arr_copy = $arr;//把数组赋值给另一个
var_dump(memory_get_usage());//打印内存
$j=1;
foreach($arr_copy as $i) {//循环遍历该数组键值查看内存情况
    // $i 是字符串
    $j += strlen($i);
}
var_dump(memory_get_usage());//打印内存
```
```
$ php ./test.php
int(405168)
int(2518816)
int(2518816)
int(2518816)
```

当$arr把值赋值给$arr_copy时,执行内存是没有明显变化的,并没有直接增加内存量,甚至在之后的foreach遍历中,也是没有增加内存的.
因为当$arr赋值给$arr_copy时,并不是在内存中复制了整个$arr的值,而是将$arr_copy的值指向了$arr,相当于在取$arr_copy的数据时,指向的还是$arr存值的内存
也就是说,<font color="red">就算我们不使用引用,php变量在传值,赋值的情况,都是指向同一个内存</font>

# 数据发生了变化

``` php
<?php
$a='cs'.time();
$b=$a;
$c=$a;
//这个时候内存占用相同,$b,$c都将指向$a的内存,无需额外占用
 
$b='cs1号';
//这个时候$b的数据已经改变了,无法再引用$a的内存,所以需要额外给$b开拓内存空间
 
$a='cs2号';
//$a的数据发生了变化,同样的,$c也无法引用$a了,需要给$a额外开拓内存空间
```