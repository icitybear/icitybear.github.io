---
title: "依赖注入" #标题
date: 2023-07-05T20:07:59+08:00 #创建时间
lastmod: 2023-07-05T20:07:59+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- 设计模式
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
# IOC 控制反转
控制反转（IOC）：当调用者需要被调用者的协助时，在传统的程序设计过程中，通常由调用者来创建被调用者的实例，但在这里，创建被调用者的工作不再由调用者来完成，而是将被调用者的创建移到调用者的外部，从而反转被调用者的创建，消除了调用者对被调用者创建的控制，因此称为控制反转。
# 依赖注入模式中，为什么用对象而不是用数组传递？
依赖注入（Dependence Injection, DI） 依赖注入是控制反转的一种设计模式。依赖注入的核心是把类所依赖的单元的实例化过程，放到类的外面去实现。依赖注入的实现离不开反射。

# 依赖注入（Dependence Injection, DI）
- 所谓的依赖注入，指将依赖的对象通过参数的形式一次性传入，<font color="red">使用时不需要显式 new 了</font>，比如把A类所依赖的B类、C类等以属性或者构造函数等方式注入A类而不是直接在A类中实例化。

- 只要不是由内部生产（比如初始化、构造函数中通过工厂方法、自行手动 new 的），而是由外部以参数或其他形式注入的，都属于依赖注入（DI） 。

- 依赖注入需要利用反射实现，比如：
``` php
<?php
class A
{
    protected $b;

    public function __constrcut(B $b)
    {
        $this->b = $b;
    }
}

// 通过控制反转容器生成 A 的实例时，会通过反射发现 A 的构造函数需要一个 B 类的实例
// 于是自动向 A 类的构造函数注入 B 类的实例
$a = IoC::make(A::class);
```

## 依赖注入的实质就是把一个类不可更换的部分和可更换的部分 分离开来，通过注入的方式来使用，从而达到解耦的目的
比如有一个 Mysql 数据库连接类如下(正常)
``` php
<?php
class Mysql {
    private $host;
    private $port;
    private $username;
    private $password;
    private $db_name;
    public function __construct(){
        $this->host = '127.0.0.1';
        $this->port = 22;
        $this->username = 'root';
        $this->password = '';
        $this->db_name = 'db';
    }
    public function connect()
    {
        return mysqli_connect($this->host, $this->username, $this->password, $this->db_name, $this->port);
    }
}

// 使用
$db = new Mysql();
$con = $db->connect();
```

<font color="red">可更换部分是数据库的配置</font>

``` php

<?php

class Configuration
{
    private $host;
    private $port;
    private $username;
    private $password;
    private $db_name;
    public function __construct($host, $port, $username, $password, $db_name)
    {
        $this->host = $host;
        $this->port = $port;
        $this->username = $username;
        $this->password = $password;
        $this->db_name = $db_name;
    }
    public function getHost()
    {
        return $this->host;
    }
    public function getPort()
    {
        return $this->port;
    }
    public function getUsername()
    {
        return $this->username;
    }
    public function getPassword()
    {
        return $this->password;
    }
    public function getDbName()
    {
        return $this->db_name;
    }
}
```

不可更换部分是Mysql数据库的<font color="red">连接操作</font>
``` php
<?php
class Mysql
{
    private $configuration;
    public function __construct(Configuration $config)
    {
        $this->configuration = $config;
    }
    public function connect(){
        return mysqli_connect($this->configuration->getHost(),$this->configuration->getUsername() ,
        $this->configuration->getPassword,$this->configuration->getDbName(),$this->configuration->getPort());
    }
}
// $config是注入Mysql的
$config = new Configuration('127.0.0.1', 'root', '', 'my_db', 22);
$db = new Mysql($config);
$con = $db->connect();
``` 
这样就完成了**配置文件和连接逻辑的分离**，使用如下,$config是注入Mysql的，这就是所谓的依赖注入。

# 总结
注入可以理解成从外面把东西打进去。因此，依赖注入模式中，要分清内和外，<font color="red">要解除依赖的类内部就是内，实例化所依赖单元的地方就是外。</font> 在外通过**构造形参**，为类内部的**抽象单元提供实例化**，达到解耦的目的，<font color="red">使下层依赖于上层，而不是上层依赖于下层。</font>

因此，依赖注入模式中，要用对象传递，通过一个实例的反射来实现，这是数组做不到的。<font color="red">(类内部的抽象单元会调用自己的方法)</font>

# 进一步研究
[就像 Laravel 框架底层服务容器-绑定接口到实现](https://laravelacademy.org/post/19434.html)