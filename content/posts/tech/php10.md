---
title: "php反射" #标题
date: 2023-07-02T12:00:16+08:00 #创建时间
lastmod: 2023-07-02T12:00:16+08:00 #更新时间
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
# Reflection 类
[官方手册](https://www.php.net/manual/zh/book.reflection.php)
- MVC里，依赖注入、控制反转，用的就是反射的特性
- 反射指在PHP运行状态中，扩展分析PHP程序，导出或提取出关于类、方法、属性、参数等的详细信息，包括注释。简而言之，就是通过对象实例反向分析，从而获取其信息。
# 利用反射实现参数绑定
通过new ReflectionMethod获取指定类指定方法需要的参数，然后判断$_GET中是否包含该参数。最后通过call_user_func_array调用类的方法并传入参数。
``` php
<?php
// $params 为参数内容
public function call($params)
{
    // 因为简易网关call_user_func_array是数组式传参
    // 所以放在一个参数数组里
    $service_name = $params['_SERVICE_'];
    $method_name  = $params['_METHOD_'];

    $class  = new ReflectionClass($service_name);
    $object = $class->newInstance();

    $method       = new ReflectionMethod($service_name, $method_name);
    $methodParams = $method->getParameters();

    $args = [];
    foreach ($methodParams as $methodParam) {
        if (isset($params[$methodParam->getName()])) {
            // 调用方法 对应名字的值
            $args[] = $params[$methodParam->getName()];
        }
    }

    return call_user_func_array([$object, $method_name], $args);
}
```
按照变量名进行参数绑定的参数必须和URL中传入的变量名称一致，但是参数顺序不需要一致。

# 代理模式
``` php
class ProxyService
{
    private $serviceClass;

    private $serviceObject;

    public function __construct($class, $params)
    {
        $this->serviceClass = $class;

        $this->serviceObject = new $class($params);
    }

    public function __get($property_name)
    {
        return $this->serviceObject->{$property_name};
    }

    public function __set($property_name, $property_value)
    {
        $this->serviceObject->{$property_name} = $property_value;
    }

    public function __call($method, $params)
    {
        $reflectionMethod = new ReflectionMethod($this->serviceClass, $method);
        $paramsName       = $reflectionMethod->getParameters(); // 参数的字段名

        $call_result = $reflectionMethod->invokeArgs($this->serviceObject, $params);

        return $call_result;
    }
}

// 实例化时候 s方法
function s() {
    // ...
    return new ProxyService($namespace . $real_class_name, $params);
}
$csv = s('Demo');
$csv->test();
```
比如 redis实例通过s方法获取后，使用scan方法
<font color="red">注意：在 __call 魔术方法中，传递的参数无法传回原始调用处,所以无法在 __call 魔术方法中传递 参数引用</font>
``` php 
class Proxy
{
    public function __call($method, $params) {
        var_export($params[0]);  // 打印 [0]
        array_push($params[0], 1);
        var_export($params[0]);  // 打印 [0, 1]
    }
}
$proxy = new Proxy();
$arr = [0];
$proxy->push($arr);
var_export($arr); // 打印 [0]
```

redis 迁移数据需要扫描旧  key
使用 scan 的时候，因为第一个值必须传递引用 reference
如果直接使用 r() 方法返回的实例， scan 的时候会报下面这个错误，<font color="red">然后 $it 一直不迭代，导致死循环</font>
[code-warinng]: [2]Parameter 1 to Redis::scan() expected to be a reference, value given exception发生错误的位置为: /data/html/privilege/vendor/haibao/xy/base/XyRedis.php 中的第124行
-  解决方案： 通过反射调用非 public 方法
  
# 通过反射调用非 public 方法

``` php 
$method = new ReflectionMethod(Benz::class, 'customMethod');
$method->setAccessible(true);

$benz = new Benz();
$method->invoke($benz);
```
![](demo.png)
