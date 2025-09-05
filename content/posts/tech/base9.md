---
title: "protobuf" #标题
date: 2025-02-28T10:51:59+08:00 #创建时间
lastmod: 2025-02-28T10:51:59+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- rpc
- gRpc
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

  
# 序列化是什么
- <font color="red">序列化通俗来说就是把内存的一段数据转化成二进制并存储或者通过网络传输，而读取磁盘或另一端收到后可以在内存中重建这段数据</font>

# idl
介绍下protobuf的idl怎么写。
protobuf最新版本为proto3
[【有道云笔记】protobuf3使用](https://note.youdao.com/s/SPZlPj6g)

# protoc
protoc就是protobuf的编译器，它把proto文件编译成不同的语言
[【有道云笔记】protoc编译器安装和使用](https://note.youdao.com/s/G3wTKpXO)

## protoc使用
protoc --help // 查看帮助命令
protoc --version // 查看版本

protoc [OPTION] PROTO_FILES

## 搜索路径

-I=PATH, --proto_path=PATH。它表示的是我们要在哪个路径下搜索.proto文件，这个参数既可以用-I指定，也可以使用--proto_path=指定。
- 多个的情况下 -I ./ -I ./third_party/, 引入第三方的时候 比如import "google/api/annotations.proto"

## 语言插件参数

--cpp_out=，--python_out=等，<font color="red">protoc支持的语言长达13种，且都是比较常见的运行help出现的语言参数，说明protoc本身已经内置该语言对应的编译插件，我们无需安装下面的语言是由google维护，通过protoc的插件机制来实现，所以仓库单独维护</font>Dart和Go

- 非内置的语言支持就得自己单独安装语言插件， --go_out=对应的是 protoc-gen-go
  
## 安装命令 protoc-gen-go
``` go
# 最新版
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
# 指定版本
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28.1

$ protoc-gen-go --version
protoc-gen-go v1.36.8

```

## protoc-gen-go
- pb文件要求必须指定go包的路径
``` go
option go_package = "xxxx";
```

- 命令来生成pb序列化相关的文件
  1. --go_out 指定go代码生成的基本路径(指定基本目录)
  2. --go_opt：设定插件参数 (可以指定多个)
``` go
protoc --proto_path=src --go_out=xxx --go_opt=paths=source_relative foo.proto bar/baz.proto

paths=import , 生成的文件会按go_package路径来生成 在--go_out目录下
    $go_out/$go_package/pb_filename.pb.go
paths=source_relative ， 就在当前pb文件同路径下生成代码 pb的目录也被包含进去了 （一般使用这个）
    $go_out/$pb_filedir/$pb_filename.pb.go
```

# protobuf 复杂的使用
[【有道云笔记】protobuf3复杂的返回结构](https://note.youdao.com/s/3wnz5C9l)

# grpc-go 插件
在google.golang.org/protobuf中，<font color="red">protoc-gen-go纯粹用来生成pb序列化相关的文件，不再承载gRPC代码生成功能。生成gRPC相关代码需要安装grpc-go相关的插件protoc-gen-go-grpc</font>

- 安装插件
``` go
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

$ protoc-gen-go-grpc --version
protoc-gen-go-grpc 1.5.1    // 1.3.0版本的比较旧

```
![alt text](image0.png)

## 生成gRPC相关代码
1. --go-grpc_out 指定grpc go代码生成的基本路径 
2. --go-grpc_opt 指定参数，并可以设置多个
``` go
protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    routeguide/route_guide.proto
```
- xxx.pb.go 包含所有类型的序列化和反序列化代码
- xxx_grpc.pb.go 定义在xxx service中的用来给client调用的接口定义,定义在 xxx service中的用来给服务端实现的接口定义

## <font color="red">需要引入第三方的protobuf</font>
项目根目录下，执行protoc命令生成代码 对应proto路径/api/vivo/vivo.proto ,**使用了import "google/api/annotations.proto"**, 使用-I 参数

![alt text](image2.png)
``` go
$ protoc -I ./ -I ./third_party/ --go_out=. --go_opt=paths=source_relative \ 
       --go-grpc_out=. --go-grpc_opt=paths=source_relative \
       ./api/vivo/vivo.proto
```
## 依赖不同的插件版本
一般是默认一个版本, 有时需要同时使用多个版本的 protoc-gen-go protoc-gen-go-grpc 来处理不同项目或遗留代码
- --plugin=protoc-gen-go=xxx  --plugin=protoc-gen-go-grpc=xxx (xxx对应环境变量) 


# github.com/golang/protobuf vs google.golang.org/protobuf

- <font color="red">google.golang.org/protobuf</font>是github.com/golang/protobuf的升级版本，v1.4.0之后github.com/golang/protobuf仅是google.golang.org/protobuf的包装

![alt text](image1.png)

# 生成的pb文件代码多版本
grpc.SupportPackageIsVersionX 当使用 protoc 生成 gRPC 代码时，生成的 .pb.go 文件会包含类似 grpc.SupportPackageIsVersion9 的声明，确保生成的代码与特定版本的 gRPC-Go 库兼容。

## SupportPackageIsVersionX
- 1-6：旧版 gRPC API 时期
- 7：重大转折点（分离 gRPC 和 Protobuf 生成器）
- 8-9：新版 API 优化期

- 有时需要同时使用多个版本的 protoc-gen-go 来处理不同项目或遗留代码。解决方案1临时覆盖安装 2使用docker 3使用sh脚本管理多版本 
- 所以一般项目管理代码 只会上传proto文件，对应的xxx.pb.go xxx_grpc.pb.go都忽略管理

## 旧版生成器会创建兼容旧gRPC的代码。
grpc.SupportPackageIsVersion9 的支持完全由代码生成工具决定。 <font color="red">即使使用最新的 gRPC 库 (v1.56.3)，如果使用旧版 protoc-gen-go (如 v1.4.x)，生成的代码仍会使用 Version7 </font> protoc-gen-go 版本（v1.25.0+）。

``` go
go get google.golang.org/grpc@v1.33.0 // 正式引入的 gRPC-Go 库
```

``` go
// 旧版
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28.0
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.3.0

// 新版
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.36.8 // latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.5.1 // latest
```

![alt text](image3.png)

# Buf 工具
使用的插件逐渐变多，插件参数逐渐变多时，命令行执行并不是很方便和直观。
- https://juejin.cn/post/7191008929986379836