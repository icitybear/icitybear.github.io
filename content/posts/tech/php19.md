---
title: "php的Stream流操作" #标题
date: 2023-07-10T15:55:35+08:00 #创建时间
lastmod: 2023-07-10T15:55:35+08:00 #更新时间
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

# 什么是流Streams

- 流Streams这个概念是在php4.3引进的，是对流式数据的抽象，用于统一数据操作，比如文件数据、网络数据、压缩数据等，以使可以共享同一套函数
- <font color="red">流就是表现出流式数据行为的资源对象。</font>
- PHP 所有 IO 都是流
**比如file_get_contents()函数即可打开本地文件也可以访问url就是这一体现。**

# 什么是包装器 wrapper

- **从理解流数据内容出发**, <font color="red">每个流都对应一种包装器</font>, 内容都是以流的方式呈现，但内容规则是不一样的，比如http协议传来的数据是流的方式，但<font color="red">只有http包装器才理解http协议传来的数据的意思,并能操作它</font>
- 默认的支持了一些协议和包装器，请用stream_get_wrappers()函数查看.也可以自定义一个包装器，<font color="red"> 用stream_wrapper_register()注册</font>
- 数据是先传给定义的包装器类对象，包装器再去操作流。
- 包装器可以嵌套，一个流外面包裹了一个包装器后，还可以在外层继续包裹包装器，这个时候里层的包装器相对于外层的包装器充当流的角色
- 每一种流打开后都可以应用任意数量的过滤器在上面，流数据会经过过滤器的处理, 既可去掉一些数据，也可以添加，还可以修改
- [php支持的协议和包装器手册](http://php.net/manual/zh/wrappers.php) 尽管RFC 3986里面可以使用:做分割符，但php只允许://
- PHP中流的形式如：://包装器的名字，内容取决于不同的包装器语法。
  - file:// — 访问本地文件系统，在用文件系统函数时默认就使用该包装器
  - http:// — 访问 HTTP(s) 网址
  - ftp:// — 访问 FTP(s) URLs
  - php:// — 访问各个输入/输出流（I/O streams）
  - zlib:// — 压缩流
  - data:// — 数据（RFC 2397）
  - glob:// — 查找匹配的文件路径模式
  - phar:// — PHP 归档
  - ssh2:// — Secure Shell 2
  - rar:// — RAR
  - ogg:// — 音频流
  - expect:// — 处理交互式的流


# 常见包装器
## 文件默认的包装器是file://,也就是说每次我访问文件系统的时候都使用了流
## PHP访问输入/输出(I/O流)的包装器。
- PHP有基本的<font color="red">php://stdin,php://stdout,php://stderr包装器</font>对应默认的I/O资源。[具体用法](https://rank.chinaz.comapi.dandelioncloud.cn/article/details/1591383459983732738)
- <font color="red">php://input流</font>，一个只读的流，流内容是post请求的数据。当我将数据放在一个post请求的body用来请求一个远程服务的时候，这个流特别好用。
  - 不需要特殊的php.ini设置。php://input不能用于enctype=multipart/form-data
  - 读取没有处置过的POST数据, 对比$HTTP_RAW_POST_DATA 内存小
  - 仅当Content-Typ为application/x-www-form-urlencod且提交方法是POST方法时，$_POST数据与php://input数据才是一致”打上引号，表示它格式不一致，内容一致）其它情况，都不一致
  - php://input读取不到GET数据。因为_GET数据作为query_path写在http请求头部（header的PATH字段），而不是写在http请求的body局部
  
# 自定义包装器
- 在用fopen、fwrite、fread、fgets、feof、rewind、file_put_contents、file_get_contents等等文件系统函数操作流时，<font color="red">数据是先传给定义的包装器类对象，包装器再去操作流。</font>
- php提供了一个类原型，只是原型而已，不是接口也不是类，不能用于继承 streamWrapper
``` php
<?php
 
/* 定义一个过滤器 */
class strtoupper_filter extends php_user_filter {
  function filter($in, $out, &$consumed, $closing)
  {
    while ($bucket = stream_bucket_make_writeable($in)) { //从流里面取出一段数据
      $bucket->data = strtoupper($bucket->data);
      $consumed += $bucket->datalen;
      stream_bucket_append($out, $bucket); //将修改后的数据送到输出的地方
    }
    return PSFS_PASS_ON;
  }
}
 
/* 注册过滤器到php */
stream_filter_register("strtoupper", "strtoupper_filter")
 or die("Failed to register filter");
 
$fp = fopen("foo-bar.txt", "w");
 
/* 应用过滤器到一个流 */
stream_filter_append($fp, "strtoupper");
 
fwrite($fp, "Line1\n");
fwrite($fp, "Word - 2\n");
fwrite($fp, "Easy As 123\n");
 
fclose($fp);
 
//读取并显示内容 将全部变为大写
readfile("foo-bar.txt");
 
```

# 流函数
[流系列函数](https://www.php.net/manual/zh/ref.stream.php)
# 流上下文


# 场景
流在平时的编程中用到并不是很多，比如使用xml-rpc时候，server端获取client数据，主要是通过php输入流input这是一种常用的场景。黑客在入侵网站的时候，也可能会用到这部分内容。
