---
title: "php的序列化和反序列化" #标题
date: 2023-07-05T18:04:42+08:00 #创建时间
lastmod: 2023-07-05T18:04:42+08:00 #更新时间
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
# 怎么理解php里面的序列化与反序列化？
序列化是将对象转换为<font color="red">字节流</font>。反序列化就是将流转换为对象。
这两个过程结合起来，可以轻松地存储和传输数据，<font color="red">在网络中可以做到跨平台、快速传输。</font>

# 两种序列化方式serialize和json

1. serialize和json序列化结果的区别在于序列化的格式。

分别用serialize/unserialize函数与json_encode/json_decode函数对对象和数组进行序列化和反序列化
  - 对象
``` php
<?php
// 对象  
$web = new stdClass;  
$web->site = 'tantengvip';  
$web->owner = 'tuntun';  
$web->age = 5; 
var_dump(serialize($web));
var_dump(unserialize(serialize($web)));
var_dump(json_encode($web));
var_dump(json_decode(json_encode($web)));

// 结果
string(87) "O:8:"stdClass":3:{s:4:"site";s:10:"tantengvip";s:5:"owner";s:6:"tuntun";s:3:"age";i:5;}"
object(stdClass)#2 (3) {
  ["site"]=>string(10) "tantengvip"
  ["owner"]=>string(6) "tuntun"
  ["age"]=>int(5)
}
string(46) "{"site":"tantengvip","owner":"tuntun","age":5}"
object(stdClass)#2 (3) {
  ["site"]=>string(10) "tantengvip"
  ["owner"]=>string(6) "tuntun"
  ["age"]=>int(5)
}
```
 - 数组
``` php
<?php
// 数组：  
$web = array();  
$web['site'] = 'tantengvip';  
$web['owner'] = 'tuntun';  
$web['age'] = 5;  
var_dump(serialize($web));
var_dump(unserialize(serialize($web)));
var_dump(json_encode($web)); 
var_dump(json_decode(json_encode($web), true));

// 结果
string(74) "a:3:{s:4:"site";s:10:"tantengvip";s:5:"owner";s:6:"tuntun";s:3:"age";i:5;}"
array(3) {
  ["site"]=>string(10) "tantengvip"
  ["owner"]=>string(6) "tuntun"
  ["age"]=>int(5)
}
string(46) "{"site":"tantengvip","owner":"tuntun","age":5}"
array(3) {
  ["site"]=>string(10) "tantengvip"
  ["owner"]=>string(6) "tuntun"
  ["age"]=>int(5)
}
```
2. serialize和json序列化的对比
   
|序列化|serialize|json|
|:---|:---|:---|
|可读性	|编码后的文本不可读，无法被其他语言的系统引用。	|变量序列化后可读性强，可以给其他系统使用。|
|编码格式	|允许非UTF-8的变量。	|只对UFT-8的数据有效。|
|处理对象	|支持除了stdClass外的其他实例。	|只对stdClass类的示例有效。|
|速度	|较小数据的情况下，serialize比json快数量级。	|大量数据的情况下，json比serialize稍差。|
|使用范围	|对象的存储使用serialize。	|与对象无关的数据存储可以使用json，如包含大量数字的数组等。|

3. serialize和json序列化对对象内成员变量和方法的处理
``` php
<?php
class Test {
    private $pri = 'pri';
    public $class = 'Test';
    public function __construct() {
        $this->class = 'Test construct';
        $this->pri = 'pri construct';
  }
  public function hello() {
        echo 'hello!';
  }
}
$test = new Test();
var_dump(serialize($test));
var_dump(unserialize(serialize($test)));
var_dump(json_encode($test));
var_dump(json_decode(json_encode($test)));
// 结果
string(86) "O:4:"Test":2:{s:9:"?Test?pri";s:13:"pri construct";s:5:"class";s:14:"Test construct";}"
object(Test)#2 (2) {
  ["pri":"Test":private]=>string(13) "pri construct"
  ["class"]=>string(14) "Test construct"
}
string(26) "{"class":"Test construct"}"
object(stdClass)#2 (1) {
  ["class"]=>string(14) "Test construct"
}
```
- **serialize**序列化和反序列化只要是类的变量都可以，但是<font color="red">类的方法</font>都无法进行序列化和反序列化。
- **json**序列化和反序列化只能序列化/反序列化类中的<font color="red">公有成员变量</font>，不能序列化/反序列化类中的私有成员变量，其类方法也无法进行序列化和反序列化。

# 对象的存储与传输
- [php对象的存储与传输](https://note.youdao.com/s/OMYOVmRl)

