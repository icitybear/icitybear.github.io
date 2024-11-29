---
title: "Test" #标题
date: 2023-06-15T11:08:38+08:00 #创建时间
lastmod: 2023-06-15T11:08:38+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- 
tags: # 标签
- demo
keywords: 
- 
description: "" #描述 每个文章内容前面的展示描述
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true #是否展示评论
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: "posts/tech/test/go2.png" #图片路径：posts/tech/文章1/picture.png
    caption: "测试封面图" #图片底部描述
    alt: ""
    relative: false

# reward: true # 打赏
mermaid: true #是否开启mermaid
---
## 图片
*文章内容的直接图片*

composer  kafka
代码风格 git风格检测 php-cs-fixer sonar
cookie session jwt 跨域 单点
加密
git
http
mysql
中间件

tool 抓包 vscode apifox iterm2  https://xiaozhou.net/learn-the-command-line-iterm-and-zsh-2017-06-23.html
业务 红包，抽奖加权，脚手架
算法写到go那
**粗体**
<font color="red">红色</font>
![](go2.png)

[百度](http://wwww.baidu.com)一下，你就知道！

+ 我是无序列表项一
+ 我是无序列表项二

1. 我是有序列表项一
2. 我是有序列表项二

`<font color="red">红色</font>`

# 测试代码块 使用高亮模板
{{< highlight python >}}
print("hello, world!")
return
{{< /highlight >}}

代码块通常缩进四个空格或一个制表符，列表项里就8个

{{< highlight html >}}
<table>
<tr>
    <th rowspan="2">姓名</th>
    <th>小明</th>
    <th>如花</th>
    <th>王尼玛</th>
</tr>
</table>
{{< /highlight >}}


## 不使用高亮块

    print("hello, world!")
    return

## 目录1
普通问题

``` php
<?php
echo phpinfo();
```

## 目录2

<div align="center">

{{<mermaid>}}
    flowchart LR
        A --> B & C --> D
{{</mermaid>}}   

</div>

## ppt

## 思维导图方案

http://www.9ong.com/082020/hugo%E6%94%AF%E6%8C%81%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE.html

## bilibili视频
{{< bilibili BV1xW4y1a7NK >}} 

## 自己博客内链接
{{< innerlink src="posts/tech/test1.md" >}}  

## 相册样式 galleries插件

{{< galleries >}}
<!-- 这里的相对目录 指的是相对于当前文件html public下的结构 不能显示gif-->
{{< gallery src="/img/docker.png" title="测试">}}
{{< gallery src="go2.png" >}}
{{< gallery src="https://www.sulvblog.cn/image/19_IMG_20220430_200901.png" >}}
{{< gallery src="/img/docker.png" title="测试">}}
{{< gallery src="go2.png" >}}
<!-- 这个html模板 目前最多展示6张，然后多余的会在最后一张轮播 第6张第一图 后面跟着数量+3  -->
{{< gallery src="https://www.sulvblog.cn/image/19_IMG_20220430_200901.png" >}}

{{< gallery src="/img/phone.png" title="测试">}}
{{< gallery src="/img/wechat.png" >}}
{{< gallery src="/img/qq.png" >}}

{{< /galleries >}}

## 也有简单的 gallery

photoswipe
figure.html gallery.html(再来个模板)
load-photoswipe.html
[https://github.com/liwenyip/hugo-easy-gallery/](https://github.com/liwenyip/hugo-easy-gallery/)