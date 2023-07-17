---
title: "php时间" #标题
date: 2023-07-11T10:07:58+08:00 #创建时间
lastmod: 2023-07-11T10:07:58+08:00 #更新时间
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

# 时间区分
GMT格林威治时间，本地时间，UNIX时间戳（单位秒数）
# 设置时区
时区php.ini  PRC,Asia/Shanghai, UTC, GMT
- 默认情况下，PHP解释显示的时间为“格林威治标准时间”，与我们本地的时间相差8个小时。
- ini_set(date.timezone, 'PRC')
- date_default_timezone_set('PRC')

**使用某些函数时，就算设置了时区，但是函数指定的还是GMT时区**

# 获取Unix 时间戳
- gmmktime，取得 <font color="red">GMT 日期</font>的 UNIX 时间戳
  - gmmktime ([ int $hour [, int $minute [, int $second [, int $month [, int $day [, int $year [, int $is_dst ]]]]]]] ) : int  返回值是格林威治标准时的时间戳
  - 空着的参数会被设为<font color="red">相应的当前 GMT 值</font>

- mktime，返回 Unix 时间戳 当前时间戳 （GMT时间1970年1月1日到现在消逝的秒数）不用自变量时，它生成当前时间的UNIX时间戳。
  - mktime ([ int $hour = date("H") [, int $minute = date("i") [, int $second = date("s") [, int $month = date("n") [, int $day = date("j") [, int $year = date("Y") [, int $is_dst = -1 ]]]]]]] ) : int
  - 任何省略的参数会被设置成<font color="red">本地日期和时间的当前值</font>

- microtime()，返回<font color="red">微秒数</font> 当前 Unix 时间戳 （<font color="red">相对于当前时区</font>）

# 获取本地时间
- strtotime 将可人为阅读的英文日期/时间字符串转换成Unix 时间戳
- time，返回当前的 Unix 时间戳
- getdate ([ int $timestamp = time() ] ) : array 取得日期／时间信息
``` php
<?php
  array(11) {
  'seconds' =>int(50)
  'minutes' =>int(20)
  'hours' =>int(2)
  'mday' =>int(11)
  'wday' =>int(2)
  'mon' =>int(7)
  'year' =>int(2023)
  'yday' =>int(191)
  'weekday' =>string(7) "Tuesday"
  'month' =>string(4) "July"
  [0] =>int(1689042050)
}
``` 

# 时间日期格式化
- date ( string $format [, int $timestamp ] ) : string
  - format输出的日期 string 格式. 同时，还可以使用 预定义日期常量 ，例如：常量 DATE_RSS 表示格式化字符串 'D, d M Y H:i:s'
  - 字母g表示小时不带前导，字母h表示小时带前导；小写g、h表示12小时制，大写G、H表示24小时制。
  - 大写T表示服务器的时间区域设置
``` php
<?php
echo date('g:i:s a'); // 5:56:57 am
echo date('h:i:s A'); // 05:56:57 AM
echo date('G:i:s'); // 14:02:26
```

- gmdate 同date()函数数完全一样，只除了返回的时间是<font color="red">格林威治标准时（GMT）</font>

- strftime ( string $format [, int $timestamp = time() ] ) : string
  - 根据区域设置格式化<font color="red">本地时间／日期</font>
  - 配合setlocale，应用此函数建立与当前环境兼容的日期字符串。
- gmstrftime 根据区域设置格式化 GMT/UTC 时间／日期

# 有效日期检测
- checkdate($month,$date,$year) 值构成一个有效日期，则该函数返回为真

# 日期操作-增减

## 获取指定时间戳的周一0点
``` php
<?php
date_default_timezone_set('PRC');
function weekFirstDateTimestamp(int $timestamp)
{
    $timestamp = $timestamp ?: time();
    $end_day = 0 == date('w', $timestamp) ? 7 : date('w', $timestamp);
    $first_date = date('Y-m-d', ($timestamp - ($end_day - 1) * 24 * 3600));

    // 等价
    $week_day = date('w', $timestamp);
    $week_day = ($week_day + 6) % 7;
    $first_date = date('Y-m-d', ($timestamp - $week_day * 24 * 3600));

    return strtotime($first_date);
}
// 当周（当前时间戳）
date('Y-m-d', strtotime("Sunday - 6 day")); 
```

## date与strtotime的坑,date自动规范化
``` php
<?php
// 没问题
echo date(“Y-m-d”,strtotime("-1 month",strtotime(‘2018-7-30’))); //结果 2018-06-30
// 有问题
echo date(“Y-m-d”,strtotime("-1 month",strtotime(‘2018-7-31’))); //结果 2018-07-01

```
1. 先做-1 month, 那么当前是07-31, 减去一以后就是06-31 <font color="red">（这里 strtotime的坑）</font>, 只要涉及到大小月的最后一天, 都可能会有这个问题 +1 -1 next last
2. 再做<font color="red">日期规范化</font>, 因为6月没有31号, 所以就好像2点60等于3点一样, 6月31就等于了7月1
**从PHP5.3开始呢, date新增了一系列修正短语, 来明确这个问题, 那就是<font color="red">”first day of” 和 “last day of”, 限定好不要让date自动”规范化”</font>**
``` php
<?php
// 错误的
var_dump(date(“Y-m-d”, strtotime("-1 month", strtotime(“2017-03-31”))));//输出2017-03-03
var_dump(date(“Y-m-d”, strtotime("+1 month", strtotime(“2017-08-31”))));//输出2017-10-01
var_dump(date(“Y-m-d”, strtotime(“next month”, strtotime(“2017-01-31”))));//输出2017-03-03
var_dump(date(“Y-m-d”, strtotime(“last month”, strtotime(“2017-03-31”))));//输出2017-03-03
// 正确的
var_dump(date(“Y-m-d”, strtotime(“last day of -1 month”, strtotime(“2017-03-31”))))//输出2017-02-28
var_dump(date(“Y-m-d”, strtotime(“first day of +1 month”, strtotime(“2017-08-31”))));// 输出2017-09-01
var_dump(date(“Y-m-d”, strtotime(“first day of next month”, strtotime(“2017-01-31”))));// 输出2017-02-01
var_dump(date(“Y-m-d”, strtotime(“last day of last month”, strtotime(“2017-03-31”))));// 输出2017-02-28
```

## date 跨年的坑 oW
- W第一个周01, 要算哪一年，统计用的话尽量使用oW
  - 使用W ISO-8601 格式年份中的第几周，每周从星期一开始（PHP 4.1.0 新加的）
  - 使用o, 如果 ISO 的星期数（W）属于前一年或下一年，则用那一年。（PHP 5.1.0 新加）
``` php
<?php
// 1577763005 时间为2019-12-31 11:30:05
echo date("W", 1577763005); // 01

echo date("oW", 1577763005); // 202001

echo date("YW", 1577763005); // 201901
```

## 秒数转化成多少时多少分多少秒
``` php
<?php
define("BJTIMESTAMP" , time()); //服务器当前时间
$expires_in	= '1439577160';//到期时间
$expires	= $expires_in - BJTIMESTAMP;
 
function time2second($seconds){
	$seconds = (int)$seconds;
if( $seconds < 86400 ){//如果不到一天
    $format_time = gmstrftime('%H时%M分%S秒', $seconds);
}else{
    if($seconds > 86400 * 365){
        $year = intval($seconds/(86400 * 365));//只取年份
    }else{
        $year = 0;
    }
    $time = explode(' ', gmstrftime('%j %H %M %S', $seconds));//Array ( [0] => 04 [1] => 14 [2] => 14 [3] => 35 )
    $format_time = ($year*365 + ($time[0]-1)).'天'.$time[1].'时'.$time[2].'分'.$time[3].'秒';
}
return $format_time;
}
echo "新浪微博授权有效期剩余: ". time2second($expires) . '<hr>';
```