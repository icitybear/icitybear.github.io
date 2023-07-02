---
title: "php运行模式,cgi,fast-cgi,php-cgi,php-fpm的关联" #标题
date: 2023-06-24T15:56:52+08:00 #创建时间
lastmod: 2023-06-24T15:56:52+08:00 #更新时间
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
    image: "posts/tech/php1/fastcgi.png" #图片路径：posts/tech/文章1/picture.png
    caption: "测试" #图片底部描述
    alt: ""
    relative: false
    hidden: true
# reward: true # 打赏
mermaid: true #自己加的是否开启mermaid
---

# cgi 协议模式
- cgi模式 通用网关接口（Common Gateway Interface）,它允许web服务器通过特定的协议与应用程序通信
- cgi模式的一次请求可以分为以下几步：
1. 用户请求
2. web服务器（apache，nginx，iis等）接收请求
3. 服务器通过cgi协议调用php，运行php文件
4. php文件处理逻辑，返回数据，php进程 销毁/回收（该次执行的php变量内存等全部回收）
5. web服务器接收数据，返回给用户，web服务器关闭连接
6. 用户接收数据，用户关闭连接
   
- php-cgi, CGI解释器，`PHP解析器`会解析php.ini文件，初始化执行环境，然后处理请求，再以规定CGI规定的格式返回处理后的结果，退出进程
- php-cgi，它是cgi协议的解析器（CGI程序），然后通过调用php的`php_execute_script函数`来解析和运行php脚本 也支持fast-cgi协议
  
# fast-cgi 协议模式 (nginx+php-fpm)
- fast-cgi是cgi模式的升级版,它像是一个<font color='red'>常驻型的cgi</font>,只要开启后,就可一直处理请求,不再需要结束进程
- 调用原理大概为:
1. web服务器fast-cgi进程管理器初始化 (比如php-fpm的master进程)->预先fork n个进程 (比如php-fpm的work进程)
2. 用户请求->web服务器接收请求->`交给fast-cgi进程管理器`->
3. fast-cgi进程管理区接收,给其中一个空闲fast-cgi进程处理->处理完成,
4. fast-cgi进程变为空闲状态,等待下次请求->
5. web服务器接收内容->返回给用户

```
注意,fast-cgi和cgi都是一种协议,开启的进程是单独实现该协议的进程
Fastcgi是CGI的升级版，一种语言无关的协议，
FastCGI是用来提高CGI程序性能的
```
![](fastcgi.png)

# php-fpm跟php和php-cgi
1. PHP-FMP 全名叫做PHP-FASTCGI Process Manager
2. php-fpm的进程包括 master(常驻服务程序)和worker进程两种
  - master进程，负责进程的调度（比如worker进程不够的时候去fork一个子进程）,还负责监听端口,一般是9000这个端口，可以在配置文件里面设置，当然，还有另外一种方式，就是通过socket，可以通过netstat -nap | grep master的进程号 查看端口信息（9000端口其实就是tcp的通信方式，而socket是说的unix socket,从效率上来说，unix socket显然是最好的，因为它是进程之间的通信，但是unix socket要保证是在一台服务器，如果是不同机器之间的通信，还是要使用tcp通信）
  - worker进程，就是进行解释php代码
3. 都能<font color='red'>解释php代码</font>，都可以fork出fast-cgi协议处理进程，提供fastCGI接口，能解释php代码(PHP解析器)，然后ngix通过网络的方式调用

4. 调用方式：
  - php(强调一下这是可执行文件，在win下叫php.exe),是cli模式调用，即用命令调用
  - php-cgi和php-fpm<font color='red'>可以通过“网络”来调用，而所使用的网络协议叫“fastCGI协议”</font>
    1. php-cgi则<font color='red'>提供了fastCGI接口</font>，fastCGI接口是一种“网络接口”，你可以通过网络的方式去调用它，比如nginx调用php-cgi可以用“fastcgi_pass 127.0.0.1:9000;”这样调用
    2. 只不过php-fpm比php-cgi高级很多

5. 缺点：win不支持php-fpm，因为<font color='red'>php-fpm是使用Linux的fork()来做的</font>，所以win下面基本上还是使用php-cgi

## php-fpm的具体功能
PHP-FPM<font color='red'>（FastCGI 进程管理器）</font>用于替换 PHP FastCGI 的大部分附加功能，对于高负载网站是非常有用的。
### 功能包括:
- 支持平滑停止/启动的高级进程管理功能;具体：对于php.ini文件的修改，<font color='red'>php-cgi进程是没办法平滑重启的</font>，有了php-fpm后，就把平滑重启成为了一种可能，<font color='red'>php-fpm对此的处理机制是新的worker用新的配置</font>
- 可以工作于不同的 uid/gid/chroot 环境下，并监听不同的端口和使用不同的 php.ini 配置文件（可取代 safe_mode 的设置）;
stdout 和 stderr 日志记录;
- 在发生意外情况的时候能够重新启动并缓存被破坏的 opcode;
- 文件上传优化支持;
- "慢日志" - 记录脚本（不仅记录文件名，还记录 PHP backtrace 信息，可以使用 ptrace或者类似工具读取和分析远程进程的运行数据）运行所导致的异常缓慢;
```
具体：比如使用php-fpm.conf配置慢日志
slowlog = /usr/local/var/log/php-fpm.log.slow
request_slowlog_timeout = 5s
```
- fastcgi_finish_request() - 特殊功能：用于在请求完成和刷新数据后，继续在后台执行耗时的工作（录入视频转换、统计处理等）;
- 动态／静态子进程产生;
- 基本 SAPI 运行状态信息（类似Apache的 mod_status）;
- 基于 php.ini 的配置文件。具体：PHP-FPM的使用非常方便，<font color='red'>配置都是在PHP-FPM.ini的文件内</font>，而启动、重启都可以从php/sbin/PHP-FPM中进行。更方便的是修改php.ini后可以直接使用`PHP-FPM reload进行加载`，无需杀掉进程就可以完成php.ini的修改加载
- <font color='red'>php-fpm.conf是PHP-FPM进程管理器的配置文件，php.ini是PHP解析器的配置文件</font>、

### 工作原理:
它的工作原理大概为:
php-fpm启动->`生成n个fast-cgi协议处理进程`->监听一个端口等待任务
用户请求->web服务器接收请求->请求转发给php-fpm->php-fpm交给一个空闲进程处理
->进程处理完成->php-fpm返回给web服务器->web服务器接收数据->返回给用户(`nginx+php-fpm 就是用的以上的方法`)

### <font color='red'>nginx + php-fpm</font>

Nginx 不支持对外部程序的直接调用或者解析，所有的外部程序（包括PHP）必须通过FastCGI接口来调用。
1. `FastCGI接口在Linux下是 socket，（这个socket可以是文件socket，也可以是ip socket）`
2. 为了调用CGI程序，还需要一个FastCGI的wrapper（wrapper可以理解为用于 <font color='red'>启动另一个程序的程序php-fpm</font>），这个 wrapper绑定在某个固定socket上，如端口或者文件socket。
3. 当Nginx将CGI请求发送给这个socket的时候，<font color='red'>通过FastCGI 接口</font>，wrapper接纳到请求，
然后派生出一个新的进程，这个进程调用解释器或者外部程序处理脚本并读取返回数据；
接着，wrapper再将返回的数据<font color='red'>通过FastCGI接口</font>，沿着固定的socket传递给Nginx；
最后，Nginx将返回的数据发送给客户端，这就是Nginx+FastCGI的整个 运作过程。
4. apache是模块模式php-cgi,SAPI, 所以无法平滑重启
   
- FastCGI的wrapper 指的就是FastCGI进程管理器。
- nginx只能解析请求，返回结果，不会管理进程，所以就出现了一些能够调度php-cgi进程的程序 

# [多版本PHP同时运行与NGINX和php联系](https://note.youdao.com/s/KcGOvImu)

# 模块模式 (apache+php运行)
apache+php运行时,默认使用的是模块模式,它把php作为apache的模块随apache启动而启动,接收到用户请求时则直接通过`调用mod_php模块`进行处理,详细内容可自行百度

# php-cli模式 (命令行模式php php.exe)
1. 除了php-cli的模式属于命令行模式,其他都定义为常规web访问模式
2. 该模式不需要借助其他程序,直接输入php xx.php 就能执行php代码
3. 命令行模式和常规web模式明显不一样的是:
- 没有超时时间
- 默认关闭buffer缓冲
- STDIN和STDOUT标准输入/输出/错误 的使用
- echo var_dump,phpinfo等输出直接输出到控制台
- 可使用的类/函数 不同
- php.ini配置的不同
  
4. php-cli模式下  while(1) 可以直接用外部的new shell脚本运行
```
#!/bin/bash

echo $$start

# 间隔时间
sleep_time=1
# 存放进程号文件目录，每个文件的文件名为当前脚本的pid，记录的内容为本次执行脚本的时间戳，该值一般不做修改。
pid_dir='/data/pid'

if [ ! -d ${pid_dir} ] ; then
    mkdir -p ${pid_dir}
fi

while true
do
    echo $(date '+%s') > ${pid_dir}/$$
    # php可执行文件使用绝对路径 /app/bin/php
    /app/bin/php ../cli.php a8f9b76a5n2f8k56d98j2 v2 HandleSlCandy run >> ../log/HandleSlCandy.log 2>&1
    sleep "$sleep_time"
done
```

# 生命周期 (php为什么无对象池)
对象池需要从php的生命周期说起，php的应用大部分都是web网站，而大部分web网站使用的都是cgi模式进行运行的，导致php生命周期跟随着请求结束而结束
在这个过程中，是根本没有对象池概念的，因为php的变量是随着用户的请求而销毁，无法把php的变量留给下一个用户进行执行（所以一些脚本使用命令行模式），
这就导致了如果用户1请求，需要new 一个对象，那么用户1请求完毕将会销毁，用户2需要重新再new一个对象，再销毁。。。。。如此反复。

## 缺点
比如：kafka推送（通过composer相关以来执行）
因PHP属于短连接请求，在使用此保进行publish时，会对kafka造成大量并发请求。而kafka本身并发能力较弱，存在打崩kafka集群的风险