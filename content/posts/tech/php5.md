---
title: "php引用和函数的值传递和引用传递和垃圾回收" #标题
date: 2023-06-27T21:57:37+08:00 #创建时间
lastmod: 2023-06-27T21:57:37+08:00 #更新时间
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

# 拷贝

涉及到对象拷贝复制，就是将一个对象的所有属性（成员变量）复制到另一个对象中，而拷贝又分为浅拷贝和深拷贝，造成这种分法的主要原因就是【值传递】和【引用传递】的不同

## 浅拷贝
若成员变量为**基本数据类型**，拷贝时为【值传递】，是两份不同的数据，改变A类中的该变量，B类不会变化
若成员变量为**引用类型**，拷贝时为【引用传递】，拷贝的是引用变量，实际上两个引用变量指向的是一个实例对象，若改变A类中的该变量，B会变化，即为【浅拷贝】
## 深拷贝 clone
深拷贝就是<font color="red">对引用类型的成员变量拷贝时，拷贝整个对象，而不是只拷贝引用</font>
嵌套的对象属性也不相互污染的拷贝才是真正相互对立的「深拷贝」。要实现这种深拷贝，得用 __clone 魔术方法, 方法里调用clone 对应的嵌套属性
# 引用
**前言**
``` php
<?php

$arr  =[3,4,5,12,8];
sort($arr);//对数组排序
var_dump($arr);//打印排序后的数组

$str ="hellow!";

$str = mb_substr($str,0,3);//剪切字符串
var_dump($str);
``` 

以上代码,分别为数组排序以及字符串截取,它们之间毫无关联,放在一起的主要原因就在于:
sort不会返回处理后的数据,而是直接修改了变量的值,mb_substr 却需要返回值来获取函数处理后的值
大多数情况下,我们封装函数,都是使用的mb_substr的方式,返回处理结果
那么,sort是怎么实现的呢?
## php引用
php引用,允许你使用多个变量访问同一部分内容,这个类似于c语言指针,但却<font color="red">不能做指针计算.通常使用&进行声明引用变量</font>,例如:

### 变量引用
``` php
<?php

$a = 1;
$b = &$a;//b的值为引用a的数据
$a=2;
var_dump($b);//b也变成了2
```
注意,`$b = &$a` 的意思不是$b指向了$a,而是<font color="red">$a和$b同时指向了同一内存</font>

### 函数返回引用
``` php
<?php
function &foo(){ //声明函数将返回一个引用值
    return $GLOBALS['a'];//返回$GLOBALS['a']
}
$GLOBALS['a']=1;
$a = foo();
//$a = &foo();
$a = 2;
var_dump($GLOBALS['a']);
```
函数返回引用跟变量引用差不多,只是函数引用将返回一个变量,然后在<font color="red">声明时增加引用</font>

### php的类引用
在php5之后,**php的类将自动返回引用**,无需增加&，自动调用:
``` php
<?php
class  test{
    public $a;
    public $b;
    public function __construct($a,$b)
    {
        $this->a= $a;
        $this->b= $b;
    }
}
$testa = new test(1,2);
//$testa = &new test(1,2);

$testb = $testa;
$testb->a = 3;
$testb->b = 4;
var_dump($testa->a,$testa->b);//3,4
```
当你new一个对象之后,不管赋值多少个变量,所有变量都将指向于同一个对象.
<font color="red">如果你需要复制一个对象不再指向同一个,请使用clone方法进行克隆对象</font>

### 销毁引用
可回去查看 <font color="red">php的垃圾回收机制</font>
``` php
<?php
$a = 1;
$b = &$a;//引用
$b = 2;//赋值

unset($b);//unset,是引用计数-1,不会影响a的值
var_dump($a);//2
``` 
可看出,<font color="red">unset只会删除变量与变量值的关联,但不会真正销毁`$a`的数据</font>,同理,如果unset($a),$b也不会受到影响

``` php
<?php
$a = 1;
$b = &$a;//引用
$b = 2;//赋值

$b = null;//直接更改内存数据为null,$a,$b都将释放原有内存
var_dump($a);//null
var_dump($b);//null
```


# 引用使用场景
在文章开头有提到过,sort是使用引用传递变量,直接修改数组数据,达到函数处理效果.
那么我们应该用引用吗?
`引用并不会加快程序执行,还可能会使代码可读性降低,但如果你有类似于sort函数,对某些数据需要处理,并且处理前的数据没有任何使用意义时,可以使用引用.`
当然,本身`php类传递`,就已经在用引用方案了,例如 `$model->where(['xx'=>'xx'])`,所以,我们可以放心使用引用,该用就用
# 函数的值传递和引用传递
## 值传递
函数参数默认以值传递方式进行传递，也就是说，我们<font color="red">传递到函数内部的实际上是变量值的拷贝，而不是变量本身</font>，还是以 add 函数为例，如果我们要实现类似 $a += $b 这种方式的求和，可以这么做：
``` php
<?php
function add(int $a, int $b): int {
   $a += $b;
   return $a;
}
```
在这段代码中，看似我们在函数体中运行 $a += $b 修改了 $a 的值，但是由于参数传递默认是值拷贝，这个赋值作用域仅限于函数体内部，在函数外部并没有真正修改 $a 的值，所以需要通过 return 语句返回 $a 才能在外部获取求和后 $a 的值，我们可以编写测试代码如下：
``` php
<?php
$a = 1;
$b = 2;
$c = add($a, $b);
printf("\$a = %d\n", $a);
printf("\$c = %d\n", $c);
上述代码的执行结果如下：
$a = 1
$c = 3
```
## 形参和实惨
可以看到 $a 的值确实没有变化，因为传递进函数的仅仅是 $a 的值拷贝而已，当然这个结果还可以从另一个角度解释，那就是<font color="red">形参（形式参数）和实参（实际参数），函数签名中的 $a、$b 仅仅是形参而已，外面定义的变量 $a、$b 才是实参</font>，为了便于标识，我们将外部调用的代码调整如下：
``` php
<?php
$m = 1;
$n = 2;
$c = add($m, $n);
printf("\$m = %d\n", $m);
printf("\$c = %d\n", $c);
```

这样，函数 add 中的 $a、$b 是形参，$m、$n 是实参就更好理解了，当我们调用函数时，实际执行了如下<font color="red">将实参赋值给形参</font>的工作：
`$a = $m;$b = $n;`
$a 后续的赋值和修改和 $m 没有任何关系。

## 引用传递  函数形参（非复合数据类型显式通过 &$a 进行声明），传递的是变量指针值（内存地址）的拷贝
如果我们想要形参 $a 的赋值和修改与实参 $m 关联起来，可不可以做到呢？当然可以，这就需要引入引用传递的概念 —— <font color="red">上面的实现传递的是值拷贝，我们把实参的指针赋值给形参</font>，这样，修改形参的值就等同于修改实参值了，因为操作的是同一个内存地址中的值，<font color="red">在 PHP 中，不支持指针的概念，可以通过引用来替代，引用和指针一样，都是通过 & 获取</font>，按照这个逻辑我们修改上述 add 方法实现如下：
``` php
<?php
function add(int &$a, int $b) {
   $a += $b;
}
// 如果要实现引用传递，需要显式通过 &$a 进行声明，这样一来，就不需要设置返回值，对变量 $a 的修改会直接同步到外部传入的实参上：
$m = 1;
$n = 2;
add($m, $n);
printf("\$m = %d\n", $m);
上述代码的执行结果是：
$m = 3
```
**<font color="red">在引用传递传递的是变量指针值（内存地址）的拷贝，所以本质上说，仍然是一种值拷贝。</font>**在对于**基本数据类型**，包括字符串、数值、布尔类型、数组而言，引用传递的时候需要**显式通过 & 进行标识**，而如果传递的对象这种**复合类型**的时候，由于**默认就是引用类型**，所以**不需要加上 & 标识**，

# 引用计数
引用计数存在的意义，就是为了使得COW可以正常运作，从而实现对内存的优化使用
## 写时复制cow
{{< innerlink src="posts/tech/php6.md" >}}  

``` php
<?php
$a='cs';
$b=$a;
$c=$a;
//这个时候内存占用相同,$b,$c都将指向$a的内存,无需额外占用
 
$b='cs1号';
//这个时候$b的数据已经改变了,无法再引用$a的内存,所以需要额外给$b开拓内存空间
 
unset($c);
//这个时候,删除$c,由于$c的数据是引用$a的数据,那么直接删除$a?
``` 
很明显,当$c引用$a的时候,删除$c,不能把$a的数据直接给删除,那么该怎么做呢?
可看出,<font color="red">unset只会删除变量与变量值的关联,但不会真正销毁`$a`的数据</font>,**引用计数-1**

## 验证引用计数

引用计数,给变量引用的次数进行计算,*当计数不等于0时*,说明这个变量已经被引用,不能直接被回收,否则可以直接回收

[PHP7中，zval结构体详解](https://www.nap6.com/qykj/202104/14128.html)

在PHP7中，zval结构体中有一个标志来决定zval是否能被引用计数。
- null,bool,int,double这些变量类型永远不会被引用计数（这个地方可能有些不太严谨，鸟哥的博客中写道PHP7中zval的类型共有18种，其中IS_LONG,IS_DOUBLE,IS_NULL,IS_FALSE,IS_TRUE不会使用引用计数）。
- object,resources,references这些变量类型总是会使用引用计数。
- array，strings这些变量类型有时会使用引用计数，有时则不会。
<font color="red">不使用引用计数的字符串类型被叫做“interned string（保留字符串）”。</font>如果你使用一个NTS(非线程安全)的PHP7来构建，通常情况下，代码中的所有字符串文字都将是限定的。这些保留字符串都是不可重复的（即，只会存在一个含有特定内容的保留字符串）。它会一直存在直到请求结束时才销毁，所以也就无需进行引用计数。如果使用了 opcache 的话，保留字符会被存储在共享内存中，在这种情况下，无法使用引用计数（因为我们引用计数的机制是非原子的）。保留字符串的伪引用计数为1。
<font color="red">对于数组来说，无引用计数的变量称为“不可变数组”。</font> 如果使用opcache，则代码中的常量数组文字将转换为不可变数组。同样的，他们存在于共享内存中，因此不得使用引用计数。不可变数组的伪引用数为2，因为它允许我们优化某些分离路径。

![](demo1.jpg)
这里的变量b后续赋值为字符串类型了

### 整型,浮点型
当变量值为整型,浮点型时,在赋值变量时,php7底层将会直接把值存储(php7的结构体将会直接存储简单数据类型),refcount将为0
``` php
<?php
<?php
$a = 1111;
$b = $a;
$c = 22.222;
$d = $c;

xdebug_debug_zval('a');
xdebug_debug_zval('b');
xdebug_debug_zval('c');
xdebug_debug_zval('d');

a: (refcount=0, is_ref=0)=1111
b: (refcount=0, is_ref=0)=1111
c: (refcount=0, is_ref=0)=22.222
d: (refcount=0, is_ref=0)=22.222
```

### 字符串，保留字符串
``` php
<?php
$a = 'aa'; // 静态字符串
$b = $a;
$c = $b;

$d = 'aa'.time();//普通字符串
$e = $d;
$f = $d;

xdebug_debug_zval('a');
xdebug_debug_zval('d');

a: (interned, is_ref=0)='aa'
d: (refcount=3, is_ref=0)='aa1688030452'

// php8的jit
$ php -d opcache.enable=1 -d opcache.jit_buffer_size=100M ./test.php 
a: (interned, is_ref=0)='aa'
d: (refcount=3, is_ref=0)='aa1688031110'
``` 
当变量值为interned string字符串型(变量名,函数名,静态字符串,类名等)时,变量值存储在静态区,内存回收被系统全局接管,引用计数为0 或者伪引用计数将一直为1(使用了opcache)

**<font color="red">当变量值为以上几种时,复制变量将会直接拷贝变量值,所以将不存在多次引用的情况</font>**

### 引用时引用计数变化
``` php
<?php
$a = 'aa';
xdebug_debug_zval('a');
$b = &$a;
$c = $b;
 
xdebug_debug_zval('a');
xdebug_debug_zval('b');
xdebug_debug_zval('c');

a: (interned, is_ref=0)='aa'
a: (refcount=2, is_ref=1)='aa'
b: (refcount=2, is_ref=1)='aa'
c: (interned, is_ref=0)='aa'

```

1. 当引用时,被引用变量($a)的value以及类型将会更改为引用类型,并将引用值指向原来的值内存地址中.
2. 之后引用变量（$b）的类型也会更改为引用类型,并将值指向原来的值内存地址,这个时候, **值内存地址被引用了2次,(原来是0)** ,所以refcount=2.
3. 而且$c并非是引用变量,所以将值复制给了$c,$c的值还是保留字符串（普通变量=引用变量）


# 垃圾回收机制

## 域回收
**php将每个运行域作为一次生命周期,每次执行完一个域,将回收域内所有相关变量**
``` php
<?php
echo "php文件的全局开始\n";
 
class A{
    protected $a;
    function __construct($a)
    {
        $this->a = $a;
        echo "类A{$this->a}生命周期开始\n";
    }
    function test(){
        echo "类test方法域开始\n";
        echo "类test方法域结束\n";
    }
//通过类析构函数的特性,当类初始化或回收时,会调用相应的方法
    function __destruct()
    {
        echo "类A{$this->a}生命周期结束\n";
        // TODO: Implement __destruct() method.
    }
}
 
function a1(){
    echo "a1函数域开始\n";
    $a = new A(1);
    echo "a1函数域结束\n";
    //函数结束,将回收所有在函数a1的变量$a
}
a1();
 
$a = new A(2);
 
echo "php文件的全局结束\n";
//全局结束后,会回收全局的变量$a
```
每个方法/函数都作为一个作用域,当运行完该作用域时,将会回收这里面的所有变量

``` php
<?php
$arr = [];
$i = 0;
while (1) {
    $arr[] = new A('arr_' . $i);
    $obj = new A('obj_' . $i);
    $i++;
    echo "数组大小:". count($arr).'\n';
    sleep(1);
//$arr 会随着循环,慢慢的变大,直到内存溢出
 
}
 
echo "php文件的全局结束\n";
//全局结束后,会回收全局的变量$arr
```
全局变量只有在脚本结束后才会回收,而在这份代码中,脚本永远不会被结束,也就说明变量永远不会回收,**$arr还在不断的增加变量,直到内存溢出**

## 内存泄漏

``` php
<?php
function a(){
    class A {
        public $ref;
        public $name;

        public function __construct($name) {
            $this->name = $name;
            echo($this->name.'->__construct();'.PHP_EOL);
        }

        public function __destruct() {
            echo($this->name.'->__destruct();'.PHP_EOL);
        }
    }

    // 实例话后 赋值给变量引用类型的数据，（引用变量）引用计数+1
    $a1 = new A('a1');
    $a2 = new A('a2');
    $a3 = new A('a3');

    // 类的互相引用 引用计数+1
    $a1->ref = $a2;
    $a2->ref = $a1;

    // unset 引用计数 -1
    unset($a1);
    unset($a2);

    echo('exit(1);'.PHP_EOL); // 脚本回收
}

a();

echo('exit(2);'.PHP_EOL);
```

**当$a1和$a2的属性互相引用时,unset($a1,$a2) 只能删除变量的引用,却没有真正的删除类的变量,这是为什么呢?**
  1. 首先,类的实例化变量分为2个步骤,
  a. 开辟类**存储空间**,用于存储类数据,
  b. 实例化一个变量(引用变量),类型为class,**值指向类存储空间**.

  2. *当给变量赋值成功后,类的引用计数为1*,同时,a1->ref指向了a2,导致a2类引用计数增加1,同时a1类被a2->ref引用,a1引用计数增加1
  3. 当unset时,只会删除类的变量引用,也就是-1,<font color="red">但是该类其实还存在了一次引用(类的互相引用)</font>
  4. 这将造成这2个类内存永远无法释放<font color="red">直到被gc机制循环查找回收,或脚本终止回收(域结束无法回收)</font>

## 脚本回收和unset手动回收
``` php
<?php
class A
{
    public $ref;
    public $name;

    public function __construct($name)
    {
        $this->name = $name;
        echo($this->name . '->__construct();' . PHP_EOL);
    }

    public function __destruct()
    {
        echo($this->name . '->__destruct();' . PHP_EOL);
    }
}

$a = new A('a');
$b = new A('b');
unset($a); // 手动回收
//a将会先回收
echo('exit(1);' . PHP_EOL);
//b需要脚本结束才会回收

a->__construct();
b->__construct();
a->__destruct();
exit(1);
b->__destruct();
```


# null和变量赋值回收
- $a=null和unset($a),作用其实都为一致,<font color="red">null将变量值赋值为null,原先的变量值引用计数-1,而**unset是将变量名从php底层变量表中清理,并将变量值引用计数-1**,唯一的区别在于,**=null,变量名还存在,而unset之后,该变量就没了**</font>
![](demo3.jpg)

- $c由于覆盖赋值,将原先A类实例的引用计数-1,导致了$c的回收,但是从程序的内存占用来说,覆盖变量并不是意义上的内存回收,只是将变量的内存修改为了其他值.内存不会直接清空.

# 使用gc_collect_cycles函数(内存泄漏时)
<font color="red">当写程序不小心造成了内存泄漏</font>,内存越来越大,可是php默认只能脚本结束后回收,那该怎么办呢?我们可以使用gc_collect_cycles函数,进行手动回收

**gc_colect_cycles 函数会从php的符号表,遍历所有变量,去实现引用计数的计算并清理内存,将消耗大量的cpu资源,不建议频繁使用**
``` php
<?php
function a(){
    class A {
        public $ref;
        public $name;
 
        public function __construct($name) {
            $this->name = $name;
            echo($this->name.'->__construct();'.PHP_EOL);
        }
 
        public function __destruct() {
            echo($this->name.'->__destruct();'.PHP_EOL);
        }
    }
 
    $a1 = new A('a1');
    $a2 = new A('a2');
 
    $a1->ref = $a2;
    $a2->ref = $a1;
 
    $b = new A('b');
    $b->ref = $a1;
 
    echo('$a1 = $a2 = $b = NULL;'.PHP_EOL);
    $a1 = $a2 = $b = NULL;
    // b为null时 因为引用计数已经0了 所以就回收了
    echo('gc_collect_cycles();'.PHP_EOL);
    echo('// removed cycles: '.gc_collect_cycles().PHP_EOL);
    //这个时候,a1,a2已经被gc_collect_cycles手动回收
    echo('exit(1);'.PHP_EOL);
 
}
a();
echo('exit(2);'.PHP_EOL);

a1->__construct();
a2->__construct();
b->__construct();
$a1 = $a2 = $b = NULL;
b->__destruct();

gc_collect_cycles();
a1->__destruct();
a2->__destruct();
// removed cycles: 4
exit(1);
exit(2);
```

# 自动回收
php内存到达一定临界值时,会自动调用内存清理,每次调用都会消耗大量的资源,可通过gc_disable 函数,去关闭php的自动gc