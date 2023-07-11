---
title: "php与rpc扩展，soap和yar" #标题
date: 2023-07-09T17:51:16+08:00 #创建时间
lastmod: 2023-07-09T17:51:16+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- php
- rpc
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

# SOAP
SOAP 即 Simple Object AccessProtocol 也就是简单对象访问协议。基于XML 和 HTTP ，其通过XML 来实现消息描述，然后再通过 HTTP 实现消息传输。
- [php_soap扩展应用](https://www.cnblogs.com/xionghao/p/7597377.html)
- [详解生成wsdl文档 soap实现web service接口服务](https://blog.csdn.net/ljl890705/article/details/79142383)

**SOA ，即Service Oriented Architecture ，中文一般理解为面向服务的架构**

# 什么是rpc
- RPC 远程过程调用 （Remote Procedure Call Protocol）——远程过程调用协议
- 采用C/S模式。请求程序就是一个客户机，而服务提供程序就是一个服务器。首先，客户机调用进程发送一个有进程参数的调用信息到服务进程，然后等待应答信息。<font color="red">在服务器端，进程保持睡眠状态直到调用信息到达为止。</font >当一个调用信息到达，服务器获得进程参数，计算结果，发送答复信息，然后等待下一个调用信息，最后，客户端调用进程接收答复信息，获得进程结果，然后调用执行继续进行

# php扩展yar(rpc框架)
- Yar 是一个轻量级, 高效的<font color="red">RPC框架</font >, 它提供了一种简单方法来让PHP项目之间可以互相远程调用对方的本地方法. 并且Yar也提供了并行调用的能力. 可以支持同时调用多个远程服务的方法.

**[php手册demo](https://www.php.net/manual/zh/yar.examples.php)**
## Yar_Server  
- 属性_executor
- Yar_Server::__construct — 创建一个HTTP RPC Server
- Yar_Server::handle — 启动HTTP RPC Server

## Yar_client
- 属性_protocol _uri _options _running
- Yar_Client::__call — 调用远程服务
- Yar_Client::__construct — 创建一个客户端实例
- Yar_Client::setOpt — 设置调用的配置

## Yar_Concurrent_Client
- 属性：_callstack; _callback; _error_callback;
- Yar_Concurrent_Client::call — 注册一个并行的服务调用
- Yar_Concurrent_Client::loop — 发送所有注册的并行调用
- Yar_Concurrent_Client::reset — 清除所有注册的服务


## 例子
1. 服务端文件访问地址 http://xxxx/operator.php
2. 被调用方operator.php
``` php
<?php
class Operator {
    xxxx
    public function add sub mul
    protected _add
    //只有公共方法可以被调用
}
$server = new Yar_Server(new Operator());
$server->handle();  //访问的时候handle
```
3. 调用方 
[一些客户端可以设置协议,参数等比如这些预定义常量](https://www.php.net/manual/zh/yar.constants.php)
``` php
<?php
//Set timeout to 1s
$client->SetOpt(YAR_OPT_CONNECT_TIMEOUT, 1000);
//Set packager to JSON
$client->SetOpt(YAR_OPT_PACKAGER, "json");
```
- 同步 Yar_client
``` php
<?php
$client = new yar_client("http://xxxx/operator.php");
var_dump($client->add(1, 2));
var_dump($client->call("add", array(3, 2))); // 回调方式

var_dump($client->_add(1, 2)); 
//私有保护方法调用不了 PHP Fatal error:  Uncaught exception 'Yar_Server_Exception' 
// with message 'call to api Operator::_add() failed' in *
```

- 注册异步调用 Yar_Concurrent_Client
``` php
<?php
function callback($ret, $callinfo) {    
    echo $callinfo['method'] , " result: ", $ret , "\n";
}

/* 注册一个异步调用 */
Yar_Concurrent_Client::call("http://xxx/operator.php", "add", array(1, 2), "callback");
Yar_Concurrent_Client::call("http://xxx/operator.php", "sub", array(2, 1), "callback");
Yar_Concurrent_Client::call("http://xxx/operator.php", "mul", array(2, 2), "callback");

/* 发送所有注册的调用, 等待返回, 返回后Yar会调用callback回掉函数 */
Yar_Concurrent_Client::loop();

/* 重置call ,否则上面的call会调用*/
Yar_Concurrent_Client::reset();
Yar_Concurrent_Client::call("http://xxx/operator.php", "sub", array(2, 1), "callback");
Yar_Concurrent_Client::loop();

// 输出
mul result: 4
sub result: 1
add result: 3
// 输出
sub result: 1
```

有个业务场景,需要本地项目去调用一个服务层的相关方法实现相应的功能,**一般情况,我可以通过普通的http的方式进行请求即可**,但是如果只是这个服务是<font color="red">内部使用</font>,那么可以使用rpc的方式进行替代.好处自不必多说,<font color="red">基于tcp传输,支持并发</font>

## 异常处理
- Yar_Server_Exception::getType — 获取异常的原始类型
- Yar_Client_Exception::getType - 客户端的
``` php
//Server.php
<?php
//服务端异常
class Custom_Exception extends Exception {};

class API {
    public function throw_exception($name) {
        throw new Custom_Exception($name);
    }
}

$service = new Yar_Server(new API());
$service->handle();

//Client.php
$client = new Yar_Client("http://host/api.php");

try {
    $client->throw_exception("client");
    //抛出服务端那边的异常
} catch (Yar_Server_Exception $e) {
    var_dump($e->getType());
    var_dump($e->getMessage());
}
```

## Yar 远程调用的实现原理
- yar client 是通过__call这个魔术方法来实现远程调用的，在Yar_client类里面并没有任何方法，当我们在调用一个不存在的方式的时候，就会执行__call方法，这个在框架中非常常见。
- Client需要远程调用的时候，先初始化数据，数据主要包括三个部分header, packager_name，request_body，然后根据配置中的方式从pack_list选择<font color="red">合适的序列化方式(msgpack，json，php)，对request_body进行序列化。</font> 然后进行传输，传输方式 同样是采用工厂方法，从已有的方式中<font color="red">选择Curl/socket方式进行传输数据</font>，整个传输以<font color="red">二进制流</font>的形式传送,传输数据的过程实际就是发送一个http请求。

服务器端监听端口，底层网络实现都利用现有的nginx, php-fpm。在PHP代码中，<font color="red">实现的地方实例化一个类</font>，然后根据request的内容，解析body，然后初始化header, 获得packager_name，<font color="red">根据packer_name解析yar_body的内容。</font>

## yar 协议
![](yar.jpg)
在 yar 中规定的传输协议如下图所示，请求体为82个字节的yar_header_t和8字节的打包名称和请求实体yar_request_t，在yar_header_t里面用body_len记录8字节的打包名称+请求实体的长度；返回体类似，只是实体内容的结构体稍微不同，在reval里面才是实际最后客户端需要的结果

## 打包器 php、json和msgpack
- 打包器类型 packager目前的实现有3种：msgpack,php,json,源码文件packager\php.c，json.c msgpack.c
- yar支持php、json和Msgpack三种打包工具，默认是php，如果要用Msgpack的话要先安装Msgpack扩展，然后配置时加上--enable-msgpack参数。但实际使用中发现，<font color="red">php的serialize</font>和msgpack的效率几乎差不太多，甚至serialize性能还要好一点点，不过serialize所占的空间要稍大一些，个人认为没有必要用msgpack。

## 传输 curl和socket

- curl的模块主要以来底层的<font color="red">CURL模块</font>，主要封装了如下方法。其中multi系列主要是解决并行请求的方案。
![](curl.jpg)
  - php_yar_curl_open 这个方法主要是用CURL创建一个连接对象。这里有个复用的机制，当options & YAR_PROTOCOL_PERSISTENT(0x1) 为true的时候，会去已经使用的连接中找一下，如果存在并且没有在用则复用。

  - php_yar_curl_send 这个函数主要是把request需要的数据，转成字符串，然后放在请求的postfield里面
  - php_yar_curl_exec 将postfield内容，通过http请求发送出去。获得返回的内容。然后按照上述内容解析出结果。

# yar升级 2.3.2版
- 必须先安装依赖msgpack
- php7.0以上 安装方式pecl，PEAR工具版本至少1.4.0以上
- YAR_OPT_CONNECT_TIMEOUT 单位毫秒, YAR_OPT_TIMEOUT 单位毫秒 以前低版本是秒
- yar的打包方式YAR_OPT_PACKAGER，设定为php，<font color="red">不同系统的环境必须设定一致，才能互相调用</font>（否则低版本调高版本能走通，高版本调低版本走不通）
![](yarext.png)

- 链接持久化, 重复建立连接有缓存，连接耗时降低一半
  - **<font color="red">yar的持久化，在进程间无法复用，也就是在 cli 脚本长时间执行的情况下有点用</font>**
  - [持久化导致的一个bug](https://note.youdao.com/s/9rbCyKl6)