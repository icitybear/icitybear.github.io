---
title: "go断点调试dlv" #标题
date: 2024-11-28T14:03:32+08:00 #创建时间
lastmod: 2024-11-28T14:03:32+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- 工具
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

# 为什么需要
Go程序出异常怎么办？pprof工具分析啊，可是如果是代码方面bug等呢？分析代码bug有时需要结合执行过程，加日志呗，可是某些异常问题服务重启之后，可能会很难复现。这时候我们可以断点调试，这样就能分析每一行代码的执行，每一个变量的结果

# 怎么用
1. 安装 go install github.com/go-delve/delve/cmd/dlv@v1.7.3

2. dlv与GDB还是比较类似的，可打印变量的值，可设置断点，可单步执行，可查看调用栈，另外还可以查看当前Go进程的所有协程、线程等；

3. 命令行和ide直接使用
``` go
//编译标识注意 -N -l ，禁止编译优化
go build -gcflags '-N -l' test.go

dlv exec test
Type 'help' for list of commands.
(dlv)
```

- [使用命令行调试的例子](https://segmentfault.com/a/1190000042659787)
# 实战
0. 本质命令行启动，需要配置对应的项目目录和参数
- -conf=/Users/chenshixiong/Documents/work/hbgo/admp-material/configs
![alt text](image4.png)
![alt text](image1.png)
1. 入口
![alt text](image0.png)
2. 打印断点的地方，控制台
![alt text](image2.png)
3. 断点的地方 堆栈 变量等待
![alt text](image3.png)