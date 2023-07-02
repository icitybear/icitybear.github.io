---
title: "PHP自动加载，命名空间，use" #标题
date: 2023-06-24T17:07:14+08:00 #创建时间
lastmod: 2023-06-24T17:07:14+08:00 #更新时间
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

当我们编写面向对象的程序时，通常是将类分别放在不同的文件中。但这样一来，当我们调用其他类的时候，则需要先手动引入该文件（否则会因为当前程序中没有该类名的类而报错）

久而久之加载的列表就会很混乱复杂，不方便管理。

# 手动加载require  require_once include include_once
文件A.php
``` php
<?php
class A{
    public function run(){
        echo "这是在A类中的方法";
    }
}
?>
``` 
文件B.php
``` php
<?php
require "A.php"; // 在这里需要先加载A.php文件
class B{
    public function run(){
        echo "这是在B类中的方法";
        $A = new A();
        $A->run();
    }
}
```
所幸，在PHP中我们可以定义<font color="red">自动加载器，自动加载需要使用的文件</font>。

#php中加载文件的几个函数的区别
- include和require是PHP中引入源文件最基本的用法，其他例如__autoload, namespace, PSR4等其实<font color="red">都是调用include,或者require而成</font>

- include有的特性require都有
加载成功会返回1，可以在被包含文件中通过return改变
当一个文件被包含时，其中所包含的代码继承了 使用包含语句所在行的变量范围，比如在函数中包含其他文件，则被包含文件中定义的变量也是函数内的作用域
<font color="red">包含是语法结构，而不是函数。不需要使用()包裹文件名</font>

- include和require的不同
require 在出错时产生 E_COMPILE_ERROR 级别的错误。换句话说将导致脚本中止。（在框架或者其他业务逻辑中，建议使用require，这样子可以提高脚本的完整性和稳定性）
<font color="red">include 只产生警告 E_WARNING ，脚本会继续运行。</font>

还有另外的include_once和require_once，这两个方法的区别跟它的普通方法一样，<font color="red">只是会检测加载的文件是否已经被加载过，如果是则不会再次加载（多了一个判断过程，稍微损耗一点性能）</font>

# 命名空间namespace
1. 没有定义命名空间，则默认为顶级命名空间, 顶级命名空间为反斜杠 \
2. 同一个PHP文件中，可以定义多个命名空间，在哪个命名空间之下，则属于哪个命名空间
3. 不但可以在命名空间下定义类，也可以在命名空间下定义常量，变量，方法等
4. 使用命名空间下的常量，变量，方法。类时，要么使用绝对命名空间，即从反斜杠\开始；要么使用相对命名空间，即相对此命名空间
   
``` php
<?php
//定义命名空间my
namespace my;
//在命名空间my下定义类My
class My{
    public function __construct(){
        echo "My";
    }
}
 
//定义命名空间test
namespace test;
new \my\My();   //这是\my\My
new My();        //这是\test\My，命名空间\test下没有类My, 将报错
new my\My();    //这是\test\my\My，命名空间\test\my下没有类My, 将报错
 
?>
```

在我们使用计算机的过程，如果想在同一个路径目录下新建两个同名的文件，将会得到错误提示，当前目录下已经存在该文件名的文件。

`在php程序中也是如此，我们没办法在同一个空间下声明两个一样类名的文件，否则会得到报错提示`
``` php
Cannot declare class A, because the name is already in use
```

``` php
<?php
class A{
    function __construct(){
        echo "第一个";
    }
}
class A{
    function __construct(){
        echo "第二个";
    }
}

```
在不同的目录中新建两个一样文件名的操作是被允许的，在程序中我们也可以通过命名空间来给代码划分目录

`将不同的代码划分到不同的空间中，两个空间的代码将相对独立开来`

新建两个文件 A1.php 和 A2.php

A1.php
``` php
<?php
namespace Siam;  // 声明命名空间使用该语法
class A{
    function __construct()
    {
        echo "这是在Siam空间下的A类";
    }
}
``` 

A2.php
``` php
<?php
// 没有声明命名空间，则是在根空间下
class A{
    function __construct()
    {
        echo "这是在根空间下的A类";
    }
}
// 此时我们可以通过普通的require引入A1.php
require "A1.php";
$A1 = new A();
```
运行A2，但是却得到结果：

这是在根空间下的A类

此时没有报错相同类名，所以可以看到使用了命名空间，将代码放到不同空间内，可以定义相同类名的类

那是因为，`虽然我们已经引入了Siam\A 但是在使用的时候没有说明我们使用的是Siam空间下的A`

当我们在某个命名空间下(如Siam)声明类的时候，该类的完整类名将是命名空间+类名如(Siam\A)

所以默认调用根的A类，我们将代码改成
``` php
require "A1.php";
$A1 = new Siam\A();
```
得到结果：这是在Siam空间下的A类

<font color="red">除了这种在调用的时候写名完整类名的方式，我们还可以`使用use提前声明，出现的所有名字为A的类，都是使用某个命名空间下的。` </font>

### use
`先有定义命名空间，其他地方才能use，这个声明使用use 关键字，通常写在文件的开头，use需要写明完整类名`
注意：use不等于require_.once或者include,
- <font color="red">use的前提是已经把文件包含进当前文件。</font>
- 其实use的还是<font color="red">类名</font>。顶便提一句，在MVC模式中，类名和文件名是相同的，所以use的时候会让不了解的人以为use后面跟的是文件名，我之前就这么以为的。
- 不同的命名空间下，有相同的类名，在同一个文件中使用,解决方案就是<font color="red">起别名</font>

新建一个文件
``` php
<?php
require "A1.php"; // 引入Siam空间下的A类文件
require "A1.php"; // 引入根空间下的A类文件
use Siam\A;  // 已经声明程序中使用的是Siam空间下的A类
$A1 = new A(); 
// 输出   这是在Siam空间下的A类
$A2 = new \A(); // 通过完整的类名，来调用根空间下的类
// 输出   这是在根空间下的A类
在使用use的时候还可以给类设置别名，防止当前脚本也有其他同名的类而导致的冲突

调用的时候值需要调用设置的别名即可

<?php
require "A1.php"; // 引入Siam空间下的A类文件
require "A1.php"; // 引入根空间下的A类文件
use Siam\A AS S_A;  // 已经声明程序中使用的是Siam空间下的A类，并且升值一个别名
$A1 = new S_A(); 
// 输出   这是在Siam空间下的A类
new A();  // 当前运行脚本没有声明namespace  所以是根空间  写的类名也不是完整类名，所以调用当前空间下的类  
// 输出   这是在根空间下的A类
$A2 = new \A(); // 通过完整的类名，来调用根空间下的类
// 输出   这是在根空间下的A类
```

# <font color="red">自动加载的原理</font>

``` php
<?php
new A();
当我们使用当前程序未定义的类时，会产生一个报错 Class 'A' not found。
```

## 1.__autoload 和 spl_autoload_register

在调用类的过程中，php会先检查当前程序内是否有该类，若没有则通过调用 __autoload函数引入该类的文件。

`__autoload ( string $class ) : void`

该方法在 php >= 7.2就被废弃了，
A.php
``` php
<?php
 
class A{
    //在构造方法中打印
    public function __construct(){
        echo "new class A";
    }
}
 
?>
```

B.php
``` php
<?php
 
//自动加载类，当使用new A()时找不到class A则将字符串A作为$className传入该__autoload方法中
function __autoload($className){
    //从className中推算出文件名，假设类名和文件名相同，且在本文件同级目录查找
    $fileName = dirname(__FILE__).DIRECTORY_SEPARATOR.$className.".php";
    //如果本文件没有指定的类，且本文件路径存在指定的文件名则包含
    if (is_file($fileName) && !class_exists($className)) {
        include $fileName;
    }else{
        die($className." not found, ". "and ". $fileName."not found");
    }
}
 
//在该文件中创建一个不再该文件的类的对象，将调用构造方法
new A();
?>
执行php B.php将输出：
F:\test>php B.php
new class A
```

至此就完成了一个简单的自动加载器的声明。我们实际的应用往往不会这么简单，这就需要我们对自动加载器的功能进一步完善才能灵活使用。

## 2.常见的加载器可以设计为：

- 定义类名与文件地址的映射

- 根据命名空间与目录层级的稳定关系追寻文件

1. 第一种加载器 定义类名与文件地址的映射
``` php
<?php
function __autoload($className){
    // 定义一个映射关系数组 如果是有使用命名空间，则要填写完整类名
    $map = [
        'A' => 'Lib/A.php',
    ];
    if ( !empty($map[$className]) ){
        require $map[$className];
    }
}
new A();
```

2. 第二种加载器 根据命名空间与目录层级的稳定关系追寻文件
``` php
<?php
function __autoload($className){
    var_dump($className);
}
new Siam\A();
运行可以得到结果string(6) "Siam\A"
```

3. 我们依旧可以像第一种自动加载器一样定义map映射，同时我们可以根据命名空间的层级创建对应的目录，这样子就可以根据命名空间找到最终储存的目录路径了
``` php
<?php
function __autoload($className){
    if (file_exists("$className.php")){
        require "$className.php";
    }
}
new Siam\A();  // 此时要把Siam\A的类放到  Siam目录下的A文件中
可以正常运行得到结果：这是在Siam空间下的A类
```

<font color="red">在不同操作系统中，目录分隔符会不同，以上代码可能不能正常运行，需要根据命名空间的\ 替换成系统的目录分隔符</font>

4. PSR规范
这种要求类文件根据命名空间存放在对应的目录层级中的约束，叫做`PSR规范`。

PSR-4规范不要求改变代码的实现方式，只建议如何使用文件系统目录结构和PHP命名空间组织代码，PSR-4规范以来PHP命名空间和文件系统目录结构查找并加载PHP类、接口和Traits。

## 3.php新版的自动加载器 spl_autoload_register (支持我们注册多个自动加载器)
我们上面介绍了__autoload方法，随着语言的发展，该方式并不能很好的为我们提供服务了。

我们有的时候会使用别人封装的类，或者将类文件放在不同的根目录中。如果此时我们使用该方法来加载，则是这样子的运行流程：
``` php
if ( 类文件是否存在A目录 ){
    加载A目录下的该类文件
} else if ( 类文件是否存在B目录) {
    加载B目录下的该类文件
}...
```
会随着系统的扩展而越来越臃肿，所以出现了一种新的注册自动加载器的方式`spl_autoload_register`

该方式可以支持我们注册多个自动加载器，<font color="red">会按照注册的顺序寻找加载类，如果中途找到则加载并停止，否则将找到结束。</font>

**该函数需要传参，可以<font color="red">为callback，类与方法名，函数名等</font>** 如
``` php
class Foo {
    static public function test($name) {
        print '[['. $name .']]';
    }
}
 
spl_autoload_register('\Foo::test'); // 自 PHP 5.3.0 起 将类的一个方法作为加载器的入口
function my_autoloader($class) {
    include 'classes/' . $class . '.class.php';
}
 
spl_autoload_register('my_autoloader');  // 将一个函数作为加载器的入口
 
// 或者，自 PHP 5.3.0 起可以使用一个匿名函数
spl_autoload_register(function ($class) {
    include 'classes/' . $class . '.class.php';
});
使用方式跟__autoload其实基本一样。只是可以更加灵活地扩展。
```

# composer的自动加载

<font color="red">除了`管理依赖包`的功能之外，`自动加载`也是composer的很重要的一个功能，</font>

我们在使用依赖包的时候，并不需要每一个文件都去加载，而是<font color="red">引入composer的入口文件即可调用所有依赖类。</font>`这就是composer已经为我们`实现了自动加载`的功能。

我们打开一个使用了composer的目录
![](https://img2022.cnblogs.com/blog/1476402/202206/1476402-20220605170629332-486598699.png)

在composer的核心中，存在着几个以autoload开头的文件，都是用来提供自动加载的功能的。
```
autoload_classmap.php 存放类与文件路径的映射

autoload_namespaces.php 存放命名空间与目录路径的映射

autoload_psr4.php 存放符合psr4规范的映射关系
```

还有其他几个是加载的逻辑的处理等等，这里就先不详细讲，主要处理是从上面几个映射关系中寻找类文件并加载。

当我们更新依赖包，新增依赖包，删除依赖包的时候。composer都会更新它维护的那几个映射文件。

<font color="red">composer也提供了我们自己定义映射的功能，我们在composer.json中可以设置配置项
当前提供PSR-0， PSR-4, classmap，files 四种加载方式的配置
注意注意注意！！！ 更新了配置文件都需要执行一下命令才能生效 composer dumpautoload</font>

## files 
如果你想要明确的指定，在每次请求时都要载入某些文件，那么你可以使用’files’ autoloading。通常作为`函数库的载入方式（而非类库）`
``` json
{
    "autoload": {
        "files": ["src/common/functions.php"]
    }
}
```
## classmap 
当我们在使用`一些不符合psr规范的类库`时，比如老版的phpqrcode，它并没有使用命名空间。

这个时候我们将这类型的类文件放在一个目录中，并使用classmap方法设置在加载类文件的时候搜索这些目录。
``` json
{
    "autoload": {
        "classmap": ["src/", "lib/"]
    }
}
```
## psr4映射设置

- PSR-4和PSR-0最大的区别 psr-0的目录路径更深
是对下划线的定义不同。PSR-4中，在类名中使用下划线没有任何特殊含义。而PSR-0则规定类名中的下划线会被转化成目录分隔符。
``` json
比如说在composer.json中我这样定义了:
{    
   "autoload": {        
       "psr-4": {            
          "church\\": "./src/"
        }
    }
}
那我使用 use church\testClass, 那就对应src/testClass.php.
使用use church\test\testClass, 那就对应src/test/testClass.php.
上面是psr-4的对应规则. 那psr-0是什么样的呢?
{    
   "autoload": {        
       "psr-0": {            
          "church\\": "./src/"
        }
    }
}
使用use church\testClass, 那就对应src/church/testClass.php.
使用use church\test\testClass, 那就对应src/church/test/testClass.php
```

现在一般都是使用PSR-4规范。
``` json
在composer.json中添加以下模块
"autoload": {
    "psr-4": {
        "Siam\\": "Lib/Siam",
        "Monolog\\": ["src/", "lib/"],  // 如果需要尝试在多个目录下寻找某个命名空间 则使用数组
    }
},
```
上面代表了Siam命名空间是对应Lib/Siam目录，以Siam为命名空间的类，会尝试从该路径中加载。
Monolog命名空间下的类可能在src目录下也可能在lib目录下，会尝试从这些路径中加载。

<font color="red">设置的命名空间必须以\\结束</font>
![](composer.jpg)
# use用途
1. 用于命名空间的别名引用// 命名空间
2. 用于trait特性能力的引入// trait
3. 匿名函数传参// 匿名函数传参  方法中要调用外部的变量是需要global的，在这里我们直接通过use()也是可以将变量传递过去的。而且这个仅限于在匿名函数中使用。