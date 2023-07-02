---
title: "php对象池" #标题
date: 2023-06-24T16:41:36+08:00 #创建时间
lastmod: 2023-06-24T16:41:36+08:00 #更新时间
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
# php为什么无对象池
对象池需要从php的生命周期说起，php的应用大部分都是web网站，而大部分web网站使用的都是cgi模式进行运行的，导致php生命周期跟随着请求结束而结束，从而没有对象池的概念
所以php脚本 会经常使用命令行模式，shell进行调用

# 对象复用
## 对象复用以及不复用的效率
那么这个时候可能会有人问？new一个对象，多大事啊！给它new不就得了！针对这个问题，我们可以来测试下new一个对象的消耗有多大

新建一个测试脚本：
{{< highlight php >}}
<?php
 
class Test
{
    protected $a;
 
    public function __construct($a)
    {
        $this->a = $a;
    }
 
    public function setA($a)
    {
        $this->a = $a;
    }
 
    public function getA()
    {
        return $this->a;
    }
}
$startTime = microtimeFloat();
for ($i = 0; $i < 10000; $i++) {
    $test = new Test($i);
    $test->getA();
}
echo "对象不复用耗时" . (microtimeFloat() - $startTime) . '秒' . PHP_EOL;
 
$startTime = microtimeFloat();
$test = new Test(0);
for ($i = 0; $i < 10000; $i++) {
    $test->setA($i);
    $test->getA();
}
echo "对象复用耗时" . (microtimeFloat() - $startTime) . '秒' . PHP_EOL;
 
 
function microtimeFloat()
{
    list($usec, $sec) = explode(" ", microtime());
    return ((float)$usec + (float)$sec);
}

测试结果
/usr/local/php-7.2.2/bin/php /home/tioncico/PhpstormProjects/test/test.php
对象不复用0.0012578964233398秒
对象复用 0.00068378448486328秒
{{< /highlight >}}

可看出，在对象复用情况下，<font color="red">效率比不复用情况快了一倍</font>，可能有人会说：缺这么性能算什么？根本看不出啊！
这是因为测试文件的类是最简单的类，如果是复杂点的，例如继承，多重继承构造函数，析构函数，以及triat等等复杂对象，花费的cpu可就不止这些了

## 为什么复用对象会比不复用快？
这个需要从2方面进行讲解
- php实例化对象步骤
如果讲php实例化的底层的话，大家可能听不懂，我也不懂底层，所以本人用通俗的方法讲解下php实例化对象需要做的事情（步骤前后顺序可能有错）<font color="red">（有兴趣的可以了解语法树，对象的序列化等等知识）</font>

1：实例化对象，检查对象属性，方法，结构等
2：属性初始化
3：对象父类属性初始化
4：构造函数初始化

  - <font color="red">可以看出，new 一个对象，所做的事跟对象的复杂度有关，比如类继承，类接口实现，检查对象继承接口等是否有错，初始化属性，调用构造函数等等</font>

- <font color="red">php 垃圾回收，同样，在回收一个对象时，需要销毁对象的所有属性，父类属性等等，以及调用析构函数等等</font>
- 如果<font color="red">对象复用</font>，这些操作将都不需要，我们只需要执行一次，即可复用

注：步骤等本人并没有详细了解，只根据本人经验进行模糊以及通俗解释

# 对象池 (php使用swoole扩展)
## 概念
- 在上面的说明中，我们已经知道了<font color="red">对象复用</font>的好处，那么如果我有2个请求同时进来呢？3个？10个？
<font color="red">（`多请求单进程处理需要php实现异步网络服务器，或者swoole协程网络服务器`）</font>

<font color="red">多个请求同时处理，一个对象是不够用的.</font>
那我们能不能先new 10个，或者100个，1000个，然后每次请求进来就分一个，标记为正在使用，其他请求不能再用这个，请求结束后标记为未使用，等待请求使用?这个操作，就是对象池。

- 顾名思义，对象池是一个池子，每次我们需要对象时从里面拿一个，用完再放回去，这样又实现了对象复用，又实现了能同时处理多个请求

## 对象池的意义 (单进程处理多个请求的情况)
上面我们可能发现了，对象池如果对象太少，比如只有10个，那10个都被人用了，岂不是第11个人没得用了？
答案是对的
- 那为什么不直接设置10000个，想多少人用就多少人用？

理论上是这样的，但是<font color="red">对象池的意义，就是限制并发的大小，防止服务器负载太高而进行宕机。</font>
例如：
    假设没有对象池，也没有对象复用，<font color="red">在传统web模式下</font>，假设进程也有100，10000个，一个请求进来需要消耗1%的cpu
当100个请求进来的时候，cpu已经为100%，勉强全部能运行
<font color="red">而出现101个请求之后，某个请求会因为cpu资源不够，处理将会变慢，直到其他请求处理好一个，腾出1%去处理新的请求</font>

如果当出现200个请求，<font color="red">cpu由于分时调度（尽量使得所有请求处理的时间尽量平均），会使得所有请求平均响应时间慢一倍</font>（如果有一个进程正常响应，那么就说明有几个请求需要慢2倍甚至更多）再到后面，将会出现只能响应少数请求，其他请求全部超时无法正常响应的宕机情况 502
 <font color="red">上面的cpu资源争夺是其一，其二是消耗内存，如果同时处理太多进程，还有可能造成服务器内存不够。</font>

对象池的意义就在于此：

- <font color="red">设定合理的对象池数量，当超出对象池数量时，让请求等待或者直接提示系统繁忙，保证其他请求进行正常响应，保证服务器的运行正常</font>

例如设置了100个对象  第101个请求进来时，使其等待3秒，3秒内如果有对象回收，则直接给101个请求使用，否则3秒后告诉该请求服务器繁忙，请稍后再试，避免出现服务器调度混乱，导致宕机

# php什么时候会用到对象池
由于对象池的特性，它只出现在`单进程`处理多个请求情况而出现（例如java的多线程同时处理），而php中大部分情况是没有的，`目前只有在swoole协程中使用较多，或者在php异步网络服务器中使用`
