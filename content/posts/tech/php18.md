---
title: "[php-cli模式]shell命令给php传参" #标题
date: 2023-07-10T11:43:14+08:00 #创建时间
lastmod: 2023-07-10T11:43:14+08:00 #更新时间
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

**有些时候需要在shell命令下把PHP当作脚本执行，比如定时任务。这就涉及到在shell命令下如何给php传参的问题，通常有三种方式传参。**

# 使用$argv or $argc参数  接收,直接使用变量
可直接程序后的 $argc参数个数 和 <font color="red"> $argv参数数组 </font> $argv文件是第0个参数
```
[root@DELL113 lee]# /usr/local/php/bin/php test.php a b c d

接收到5个参数Array
(
    [0] => test.php
    [1] => a
    [2] => b
    [3] => c
    [4] => d
)
``` 

- 区分 func_get_args 返回参数列表数组

# 使用getopt函数

``` php
<?php
/**
 * 使用 getopt函数
 */
 
$param_arr = getopt('a:b:');
print_r($param_arr);
```

```
[root@DELL113 lee]# /usr/local/php/bin/php test.php -a 345
Array
(
    [a] => 345
)
[root@DELL113 lee]# /usr/local/php/bin/php test.php -a 345 -b 12q3
Array
(
    [a] => 345
    [b] => 12q3
)
[root@DELL113 lee]# /usr/local/php/bin/php test.php -a 345 -b 12q3 -e 3322ff
Array
(
    [a] => 345
    [b] => 12q3
)
```
# 提示用户输入(标准输入/输出)

``` php
<?php
fwrite(STDOUT,'请输入您的博客名：');
echo '您输入的信息是：'.fgets(STDIN);
```
```
[root@DELL113 lee]# /usr/local/php/bin/php test.php 

请输入您的博客名：hello world
您输入的信息是：hello world
```