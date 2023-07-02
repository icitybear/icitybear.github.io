---
title: "php错误和异常" #标题
date: 2023-07-02T18:21:36+08:00 #创建时间
lastmod: 2023-07-02T18:21:36+08:00 #更新时间
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

# 错误级别
**[官方文档](https://www.php.net/manual/zh/errorfunc.constants.php)**
- 调用 error_reporting 设置错误报告级别, 通过位运算error_reporting(E_ALL ^ E_NOTICE);
- PHP 全局配置文件 php.ini 中默认的错误报告级别
``` shell
$ php -i | grep error_reporting
error_reporting => 32767 => 32767
xdebug.force_error_reporting => 0 => 0
```
# 自定义错误处理器 set_error_handler
通过 set_error_handler 函数指定自定义错误处理器对错误进行处理，自定义处理器通常是个自定义函数，在这个函数中，我们可以自定义不同级别错误的处理逻辑
- 如果存在该方法，相应的error_reporting()就不能在使用了。所有的错误都会交给自定义的函数处理。
- 该函数只能捕获系统产生的一些Warning、Notice级别的错误
``` php

<?php

// error_reporting(E_ALL);  // 报告所有错误（默认配置）
// error_reporting(E_ALL ^ E_WARNING); // 排除警告的时候，错误就不会报
set_error_handler("myErrorHandler");

$content = file_get_contents('https://xueyuanjun.com/error');
var_dump($content);

/**
 * 自定义错误处理器
 * @param $errno int 错误级别
 * @param $errstr string 错误信息
 * @param $errfile string 错误文件
 * @param $errline  int   错误行号
 */
function myErrorHandler($errno, $errstr, $errfile, $errline)
{
    // 该级别错误不报告的话退出
    if (!(error_reporting() & $errno)) {
        return;
    }

    switch ($errno) {
        case E_ERROR:
            echo "致命错误类型: [$errno] $errstr\n";
            break;
        case E_WARNING:
            echo "警告错误类型: [$errno] $errstr\n";
            break;
        case E_NOTICE:
            echo "一般错误类型: [$errno] $errstr\n";
            break;
        default:
            echo "未知错误类型: [$errno] $errstr\n";
            break;
    }
}
// 不屏蔽warning的情况，自带报错
$ php ./test.php
PHP Warning:  file_get_contents(https://xueyuanjun.com/error): Failed to open stream: HTTP request failed! HTTP/1.1 404 Not Found
 in /Users/chenshixiong/Documents/work/haibao/tq-trade/social-trade/test.php on line 6
PHP Stack trace:
PHP   1. {main}() /Users/chenshixiong/Documents/work/haibao/tq-trade/social-trade/test.php:0
PHP   2. file_get_contents($filename = 'https://xueyuanjun.com/error') /Users/chenshixiong/Documents/work/haibao/tq-trade/social-trade/test.php:6
xxxx
/Users/chenshixiong/Documents/work/haibao/tq-trade/social-trade/test.php:7:
bool(false)

// 屏蔽自带的报错，无自定义处理器
$ php ./test.php
/Users/chenshixiong/Documents/work/haibao/tq-trade/social-trade/test.php:7:
bool(false)

// 自定义报错
$ php ./test.php               
警告错误类型: [2] file_get_contents(https://xueyuanjun.com/error): Failed to open stream: HTTP request failed! HTTP/1.1 404 Not Found
```

# 将错误报告写入日志
错误报告默认输出到<font color="red">标准输出 STDOUT </font>中了，我们还可以通过 <font color="red">error_log 函数</font>将其输出到指定日志文件
- 日志函数 error_log 中，第一个参数是错误消息，第二个参数是写入目标（3 表示指定文件，1 表示邮箱，0 表示系统日志），第三个参数即目标值，这里是自定义的日志文件

可以看到 STDOUT 中不再输出日志，而是写入到日志文件

# error异常
- Error 异常和 Exception 类并不是父子关系，而是兄弟关系，所以不能通过 Exception 捕获 Error 异常
- 在 PHP 7 中，大多数错误被作为 Error 异常抛出，这种 Error 异常可以像 Exception 那样被捕获，如果没有对 Error 异常进行捕获，则调用全局异常处理器（通过 set_exception_handler 函数注册）处理，如果全局异常处理器也没有注册，则按照传统错误报告方式处理
## 捕获错误
``` php
<?php

// error_reporting(E_ALL);  // 报告所有错误（默认配置）
// error_reporting(E_ALL ^ E_WARNING);
// set_error_handler("myErrorHandler");

try {
    $content = file_get_contents('https://xueyuanjun.com/error');
} catch (Error $error) { // 错误类
    var_dump($error);
}
var_dump($content);
```

# 向用户显示错误报告和 Error 异常
设置 display_errors 选项决定是否向用户显示错误报告和 Error 异常，该配置默认在 PHP 配置文件中全局设置，你也可以通过 ini_set 在运行时设置
**ini_set('display_errors', 0);** 默认为 1，表示显示用户级错误，设置为 0 则表示不显示用户级错误.不处理error_log函数

# 错误与异常
错误指的是致命错误（Fatal Error，比如编译错误和语法错误），出现运行时错误后，程序应该无法继续往后执行，需要执行一些清理工作并记录日志后退出当前处理流程。
而异常指的是程序中出现的可预测的、可恢复的中轻度问题，比如数空对象引用、文件不存在、除数为零、数组越界等，当程序运行时出现异常后，我们可以对其进行捕获，或者抛给上层的业务代码处理，和错误报告类似，如果通过 <font color="red">set_exception_hanlder 函数</font>定义了全局异常处理器，则所有未处理异常会集中到这里处理，如果没有定义任何处理异常的代码，最终会抛出一个 Fatal Error（也就是说，所有未处理异常都会被当作错误进行兜底处理）。程序出现异常后，应该可以继续往后执行。

- PHP5之前 中只有错误，没有异常，所以你可以看到那么多的错误级别，比如 Notice、Warning、Deprecated 这些中轻度错误，实际上完全可以通过异常进行处理。
- set_exception_hanlder, 用在没有用try/catch块来捕获的异常
- Error 和 Exception 类又都实现了 Throwable 接口。
- Exception 类中的很多方法定义前面都有一个 final 关键字，通过该关键字修饰的方法不能被子类重写
# 层次结构
PHP 7 中，所有错误都归属于 Error 类，所有异常都归属于 Exception 类，两者是并列关系
![](error.jpg) 
## 捕获异常
- 通过多个 catch 语句进行捕获
- 通过 throw 关键字即可抛出异常
- 如果你不知道抛出的异常类型是什么，可以通过 Exception 基类捕获（或者其他父级异常类）
- 通过添加 finally 语句块定义一个兜底逻辑
``` php
<?php
$exit = false;
try {
    $val = getItemFromBook($book, 'desc');
} catch (InvalidArgumentException $exception) {
    echo $exception->getMessage() . PHP_EOL;
    $exit = true;
} catch (OutOfBoundsException $exception) {
    echo $exception->getMessage() . PHP_EOL;
    $exit = true;
    // throw $exception; // 捕获到异常后不进行处理，直接抛出，交给上一层调用代码进行进一步处理
} finally {
    $exit ? exit() : var_dump($val);
}
``` 
## 全局异常捕获 set_exception_hanlder

## 自定义异常类
自定义的异常类，只需要继承自 Exception 基类或者其子类即可
# register_shutdown_function