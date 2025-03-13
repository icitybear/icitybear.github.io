---
title: "授权oauth2.0" #标题
date: 2024-09-05T14:50:50+08:00 #创建时间
lastmod: 2024-09-05T14:50:50+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- web技术
- 网络编程
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

# 是什么
OAuth 就是一种授权机制。数据的所有者告诉系统，同意授权第三方应用进入系统，获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。
- oauth2.0 标准化协议  授权协议规范
- 是一种授权机制，主要用来颁发令牌（token）

# oidc
- b站视频 [OAuth2 OIDC他们之间的关系以及相互关系](https://www.bilibili.com/video/BV1K84y1Q71D)

- 正常oauth2.0 先拿临时授权码（时效短），带上授权码访问 获得access_token和refresh_token,以前返回的结构体还有其他用户信息，先规定直接用id_token
- open id的升级版 open id connect。 强化oauth2.0 用来补充认证
- id_token 也是jwt (一般是携带 第三方账号信息)
``` json
{"access_token":"ACCESS_TOKEN","token_type":"bearer","expires_in":2592000,"refresh_token":"REFRESH_TOKEN","scope":"read","uid":100101,"info":{...}}
// 新版的直接 id_token 是jwt包含
```

# 常见4种授权方式 b授权给a
1. 客户端凭证授权   直接给第三方软件token， 提供api直接请求（权限太大，一般内部自己微服务）是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。
2. 密码授权流程  第三方软件使用用户的账号密码登录后给token，相当于开了一个不需要验证码的后门。密码会泄漏个第三方软件  （一般使用系统都是个人的）
3. 隐式授权流程， 是用户自已登录后给token，访问回调地址，直接通过url明文给的access_token。 但是这种token泄露的风险大 (令牌位置URL锚点，而不是查询字符串)
4. 授权码授权， 用户自已登录后,（依据appId scope 生成）访问回调地址，给一个换token一次性码（临时授权码） 。后续第三方根据约定请求系统授权接口获得access_token和refresh_token（常用，后续还有OIDC id_token）

- 不管哪一种授权方式，第三方应用申请令牌之前，都必须先到系统备案，说明自己的身份，然后会拿到两个身份识别码：<font color="red">客户端 ID（client ID）和客户端密钥（client secret）</font>。这是为了防止令牌被滥用，没有备案过的第三方应用，是不会拿到令牌的。
 
# 授权码（authorization code）方式
指的是第三方应用先申请一个授权码，然后再用该码获取令牌。这种方式是<font color="red">最常用的流程，安全性也最高，它适用于那些有后端的 Web 应用。</font>授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄漏。
1. A 网站提供一个链接，用户点击后就会跳转到 B 网站，授权用户数据给 A 网站使用。
``` go
// 下面就是 A 网站跳转 B 网站的一个示意链接。
https://b.com/oauth/authorize?response_type=code&client_id=CLIENT_ID&redirect_uri=CALLBACK_URL&scope=read

// response_type参数表示要求返回授权码（code）
// client_id参数让 B 知道是谁在请求
// redirect_uri参数是 B 接受或拒绝请求后的跳转网址
// scope参数表示要求的授权范围（这里是只读
```
2. 用户跳转后，B 网站会要求用户登录，然后询问是否同意给予 A 网站授权。用户表示同意，这时 B 网站就会跳回redirect_uri参数指定的网址。跳转时，会传回一个授权码
   - 临时授权码，是短期有效的
3. 在后端，向 B 网站请求令牌。根据约定请求B授权接口, B 网站收到请求以后，就会颁发令牌。<font color="red">具体做法是向redirect_uri指定的网址(A网)，发送一段 JSON 数据。获得access_token和refresh_token（常用，后续还有OIDC id_token）</font>

# 隐藏式
- 这种方式没有授权码这个中间步骤
1. A 网站提供一个链接，要求用户跳转到 B 网站，授权用户数据给 A 网站使用。
``` go
https://b.com/oauth/authorize?response_type=token&client_id=CLIENT_ID&redirect_uri=CALLBACK_URL&scope=read
// response_type参数为token，表示要求直接返回令牌
```
2. 用户跳转到 B 网站，登录后同意给予 A 网站授权。这时，B 网站就会跳回redirect_uri参数指定的跳转网址，<font color="red">并且把令牌作为 URL 参数</font>，传给 A 网站
``` go
https://a.com/callback#token=ACCESS_TOKEN
```
   - 令牌位置URL锚点，而不是查询字符串，这是因为 OAuth 2.0 允许跳转网址是 HTTP 协议，因此存在"中间人攻击"的风险，而浏览器跳转时，锚点不会发到服务器，就减少了泄漏令牌的风险。

# 密码式
1. A 网站要求用户提供 B 网站的用户名和密码。拿到以后，A 就直接向 B 请求令牌
``` go
https://oauth.b.com/token?grant_type=password&username=USERNAME&password=PASSWORD&client_id=CLIENT_ID
// grant_type参数是授权方式，这里的password表示"密码式"，username和password是 B 的用户名和密码。
```
2. B 网站验证身份通过后，直接给出令牌。注意，<font color="red">这时不需要跳转，而是把令牌放在 JSON 数据里面，作为 HTTP 回应，A 因此拿到令牌</font>

# 凭证式
- 适用于没有前端的命令行应用
1. A 应用在命令行向 B 发出请求
``` go
https://oauth.b.com/token?grant_type=client_credentials&client_id=CLIENT_ID&client_secret=CLIENT_SECRET
// grant_type参数等于client_credentials表示采用凭证式，client_id和client_secret用来让 B 确认 A 的身份
```
然后直接使用令牌，头信息携带token
curl -H "Authorization: Bearer ACCESS_TOKEN" \"https://api.b.com"
- 令牌更新
``` go
https://b.com/oauth/token?grant_type=refresh_token&client_id=CLIENT_ID&client_secret=CLIENT_SECRET&refresh_token=REFRESH_TOKEN
// grant_type参数为refresh_token表示要求更新令牌，client_id参数和client_secret参数用于确认身份，refresh_token参数就是用于更新令牌的令牌。
```
B 网站验证通过以后，就会颁发新的令牌。

# 案例
- 案例：[【有道云笔记】微信开放平台开发——网页微信扫码登录（OAuth2.0）](https://note.youdao.com/s/CbrEn8Xe)
- A 网站允许 GitHub 登录 https://www.ruanyifeng.com/blog/2019/04/github-oauth.html
- 投放管家授权