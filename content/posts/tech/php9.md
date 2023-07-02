---
title: "php回调call_user_func和call_user_func_array" #标题
date: 2023-07-02T11:38:05+08:00 #创建时间
lastmod: 2023-07-02T11:38:05+08:00 #更新时间
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
回调函数就是在主进程执行当中,突然跳转到预先设置好的函数中去执行的函数.
你到一个商店买东西，刚好你要的东西没有货，于是你在店员那里留下了你的电话，过了几天店里有货了，店员就打了你的电话，然后你接到电话后就到店里去取了货。
在这个例子里，
- 你的电话号码就叫回调函数（函数名，匿名函数）
- 你把电话留给店员就叫登记回调函数 （先声明或传入）
- 店里后来有货了叫做触发了回调关联的事件（条件）
- 店员给你打电话叫做调用回调函数 （调用 回调函数）
- 你到店里去取货叫做响应回调事件。（回调函数  做了啥）

# demo 关键字callable 
首先定义了一个插入数据的函数,定义了一个1001条数据的数组 然后调用了action函数,当遍历数组满足条件时,则执行设定好的回调函数进行插入数据
``` php
<?php
//登记回调函数
function insert(int $i):bool {
    echo "插入数据{$i}\n";//模拟数据库插入//响应回调事件
    return true;
}
$arr = range(0,1000);//模拟生成1001条数据
function action(array $arr, callable $function)
{
    foreach ($arr as $value) {
        if ($value % 10 == 0) {//当满足条件时,去执行回调函数处理//触发回调
            call_user_func($function, $value);//调用回调事件
        }
    }
}
action($arr,'insert'); // 函数字符串:

//匿名函数
// action($arr,function($i){
//     echo "插入数据{$i}\n";//模拟数据库插入
//     return true;
// });

```

# 类静态方法
``` php
<?php
action($arr,'A::insert');
action($arr,['A','insert']); //一般选这个
```
# 类方法
``` php
action($arr,[$a,'insert']); // $a类实例
```
# call_user_func_array
<font color="red">call_user_func()是利用回调函数处理字符串，call_user_func_array是利用回调函数处理数组。不同在于传参方式，前者是字符串形式，如果是多个参数的话，还是需要以列表的形式列出。后者是数组形式。</font>
``` php
// 字符串传参
call_user_func($func_name, 1, 2); // 3
// 数组式传参
call_user_func_array($func_name, [1, 2]); // 3
// 类静态方法
call_user_func(['A', $func_name], 1, 2);
call_user_func_array(['A', $func_name], [1, 2]);
// 类方法
call_user_func([$a, $func_name], 1, 2);
call_user_func_array([$a, $func_name], [1, 2]);
```
![](callback.jpg)