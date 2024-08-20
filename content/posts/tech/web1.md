---
title: "访问权限控制casbin" #标题
date: 2024-08-19T11:45:22+08:00 #创建时间
lastmod: 2024-08-19T11:45:22+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- go包
- web技术
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

# 简介
Casbin是一个强大的、高效的开源访问控制框架，其权限管理机制支持多种访问控制模型。
- 支持ACL，RBAC，ABAC等常用的访问控制模型。 支持resultFulAPI,支持优先级等等
  - https://casbin.org/zh/docs/supported-models/
  - ACL 缺点 每次新增一个用户，都要把他需要的权限重新设置一遍
  - <font color="red">RBAC 权限的增加和删除都通过角色来进行, 通过配置role_definition。一个用户是可以有多个角色的</font>

1. ACL (访问控制列表)
2. 带有超级用户的ACL
3. 无用户的ACL：这对于没有身份验证或用户登录的系统特别有用。
4. 无资源的ACL：在某些情况下，目标是一种资源类型，而不是单个资源。 可以使用像"write-article"和"read-log"这样的权限。 这并不控制对特定文章或日志的访问。
5. RBAC (基于角色的访问控制)
6. 带有资源角色的RBAC：用户和资源同时可以拥有角色（或组）。
7. 带有域/租户的RBAC：用户可以为不同的域/租户拥有不同的角色集。
8. ABAC (基于属性的访问控制)：可以使用类似"resource.Owner"的语法糖来获取资源的属性。
9. RESTful：支持像"/res/*"，"/res/:id"这样的路径，以及像"GET"，"POST"，"PUT"，"DELETE"这样的HTTP方法。
10. 拒绝优先：同时支持允许和拒绝授权，其中拒绝优先于允许。
11. 优先级：策略规则可以设置优先级，类似于防火墙规则。

# 文档
[b站视频](https://www.bilibili.com/video/BV18bePeuEBu)

# 简单使用
- casbin的核心是一套基于PERM metamodel(Policy,Effect,Request,Matchers)的DSL

创建一个Casbin决策器需要有
- model.pml 模型文件(配置文件)
  - 对应的字母是写死的 
```
//配置
[request_definition]        //请求定义
r = sub, obj, act           // 一个请求有三个标准元素，请求主体，请求对象，请求操作
[policy_definition]         // 策略定义，也就是*.cvs文件 p 定义的格式
p = sub, obj, act           // 在 policy.csv 中每⼀⾏定义的 policy_rule 就必须和这个属性⼀⼀对应
[role_definition]           //组定义（角色定义），也就是*.cvs文件 g 定义的格式
g = _, _
[policy_effect]              // Effect ⽤来判断如果⼀个请求满⾜了规则，是否需要同意请求。
e = some(where (p.eft == allow)) // some表示括号中的表达式个数大于等于1就行

[matchers]                 // 用来判断请求是否匹配某个规则
m = g(r.sub, p.sub) && keyMatch(r.obj, p.obj) && (r.act == p.act || p.act == "*")
//请求用户与满足*.cvs p(策略)且满足g(组规则)且请求资源满足p(策略)规定资源
// 自定义写法 root的超级管理员
[matchers]
m = r.sub == p.sub && r.obj == p.obj && r.act == p.act || r.sub == "root"
// 正则匹配
[matchers]
m = r.sub == p.sub && keyMatch(r.obj, p.obj) && regexMatch(r.act, p.act)
``` 
- policy.csv 策略文件(规则集)
``` csv
//policy策略
p, alice, /dataset1/*, GET         //alice 用户有对 method为GET路径满足 /dataset1/*的访问权限 下面同理
p, alice, /dataset1/resource1, POST
p, bob, /dataset2/resource1, *
p, bob, /dataset2/resource2, GET
p, bob, /dataset2/folder1/*, POST
p, cathrin, /dataset2/resource2, GET
p, dataset1_admin, /dataset1/*, *
g, cathrin, dataset1_admin  //cathrin用户属于dataset1_admin组，也就是dataset1_admin能访问的cathrin都能访问，反之不然 RBAC
```
- 创建一个Casbin决策器
## demo
model.pml
```
[request_definition]
r = sub, obj, act 

[policy_definition]
p = sub, obj, act

[matchers]
m = r.sub == p.sub && r.obj == p.obj && r.act == p.act

[policy_effect]
e = some(where (p.eft == allow)) // some表示括号中的表达式个数大于等于1就行
```
- 以用户 fengfeng 用GET请求 /api/users接口为例：请求主体就是 fengfeng请求对象就是 /api/users 请求操作就是 GET
policy.csv
``` csv
p, zhangsan, /index, GET
p, zhangsan, /home, GET
p, zhangsan, /users, GET
p, zhangsan, /users, POST
p, wangwu, /index, GET
p, wangwu, /home, GET
```
  
``` go
package main

import (
  "fmt"
  "github.com/casbin/casbin/v2"
  "log"
)

func check(e *casbin.Enforcer, sub, obj, act string) {
  // 决策器 判断权限
  ok, _ := e.Enforce(sub, obj, act)
  if ok {
    fmt.Printf("%s CAN %s %s\n", sub, act, obj)
  } else {
    fmt.Printf("%s CANNOT %s %s\n", sub, act, obj)
  }
}

func main() {
  // 创建一个Casbin决策器
  e, err := casbin.NewEnforcer("./model.pml", "./policy.csv")
  if err != nil {
    log.Fatalf("NewEnforecer failed:%v\n", err)
  }

  check(e, "zhangsan", "/index", "GET")
  check(e, "zhangsan", "/home", "GET")
  check(e, "zhangsan", "/users", "POST")
  check(e, "wangwu", "/users", "POST")
}
// 输出
// zhangsan CAN GET /index
// zhangsan CAN GET /home
// zhangsan CAN POST /users
// wangwu CANNOT POST /users   策略只有GET方式
```

## 决策器 临时增加/删除规则（规则集=》策略）
``` go
// 增加
e.AddPolicy("wangwu", "/users", "POST")
e.SavePolicy()
// 删除
e.RemovePolicy("wangwu", "/users", "POST")
e.SavePolicy()
```

## 角色RBAC
``` go
[request_definition]
r = sub, obj, act
[policy_definition]
p = sub, obj, act
[role_definition] // 角色的配置
g = _, _
[policy_effect]
e = some(where (p.eft == allow))
[matchers]
m = g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act // 

p, admin, /index, GET //p开头配置策略
p, admin, /home, GET
p, admin, /users, GET
p, admin, /users, POST
p, yunwei, /index, GET
p, yunwei, /home, GET
g, zhangsan, admin // g开头 配置角色组
g, wangwu, yunwei
//-------以上配置项----//
check(e, "zhangsan", "/index", "GET")
check(e, "zhangsan", "/home", "GET")
check(e, "zhangsan", "/users", "POST")
check(e, "wangwu", "/users", "POST")

// zhangsan CAN GET /index // 角色是admin
// zhangsan CAN GET /home
// zhangsan CAN POST /users
// wangwu CANNOT POST /users // 角色是yunwei
```
### 增加新角色
``` go
e.AddPolicy("kuaiji", "/users", "GET")
e.SavePolicy()
```
### 管理用户和角色对应关系
``` go
e.AddRoleForUser("zhangsan", "kuaiji")
e.SavePolicy()
// 删除用户角色关系
e.RemoveGroupingPolicy("zhangsan", "kuaiji")
e.SavePolicy()
```

# gorm接入Casbin
- 本质就是把规则集（策略配置放到mysql表里）

go get github.com/casbin/gorm-adapter/v3 包
``` go
gormadapter "github.com/casbin/gorm-adapter/v3"

global.DB = core.InitGorm() // 初始化数据库
// NewAdapterByDB构造一个gorm的适配器适配 策略配置实例
a, _ := gormadapter.NewAdapterByDB(global.DB)
// NewModelFromFile 从模型文件配置获取 模型实例
m, err := model.NewModelFromFile("./testdata/model.pml") // 模型配置
if err != nil {
  logrus.Error("字符串加载模型失败!", err)
  return
}
//  决策器这里需要使用 使用缓存方法 参数分别是 模型实例和策略配置实例
e, _ := casbin.NewCachedEnforcer(m, a)
e.SetExpireTime(60 * 60) // 过期时间
_ = e.LoadPolicy() // 刷新策略

// 策略 
e.AddPolicy("admin", "/api/users", "GET")
// 增加用户角色关系
e.AddRoleForUser("zhangsan", "admin")
check(e, "zhangsan", "/api/users", "GET")
// 删除用户角色关系
e.RemoveGroupingPolicy("zhangsan", "admin")
// 删除策略
e.RemovePolicy("admin", "/api/users", "GET")
check(e, "zhangsan", "/api/users", "GET")
// 保存策略
e.SavePolicy()
check(e, "zhangsan", "/api/users", "GET")

```