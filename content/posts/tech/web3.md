---
title: "会话方案-cookie,session和token,jwt" #标题
date: 2024-08-20T14:12:31+08:00 #创建时间
lastmod: 2024-08-20T14:12:31+08:00 #更新时间
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

# 会话凭证技术
- cookie字符串很多都是账号与密码的简单组合。随着大家注意到安全问题，大家就着手对cookie做些变化，从简单变化到使用哈希算法加密，逐渐演变成了session。
- cookie信息载体（浏览器的存储技术 localstorage）
- session需要服务存放session_id与用户信息的映射表 （可放mysql会有负载压力问题）通常放redis （后续因为跨平台问题）
- token避免服务端存放信息映射吃服务器内存,服务端要支持CORS(跨来源资源共享)策略
- JWT是token的一种，适用分布式，无状态。也算是一种更高级的Cookie方案吧？如果Cookie时代有人手动把Cookie分为两部分一部分放用户ID，一部分放签名防伪造；完全就像JWT的雏形了.

- b站视频: [Cookie、Session、Token、JWT一次性讲完](https://www.bilibili.com/video/BV18u4m1K7D4)

- [【有道云笔记】会话认证深入理解（cookie session token jwt）](https://note.youdao.com/s/HUWN0mWl)
- [【有道云笔记】HTTP跨域详解和解决方式 CORS和JSONP](https://note.youdao.com/s/9bwH7VT5)
  - b站视频: [彻底搞懂CORS（跨域资源共享）](https://www.bilibili.com/video/BV13z4y1F717)
- [【有道云笔记】xss攻击和csrf攻击的定义及区别](https://note.youdao.com/s/NzF6oaEP)
  
# token
## 基于token的鉴权机制
基于token的鉴权机制类似于http协议也是无状态的，它不需要在服务端去保留用户的认证信息或者会话信息。这就意味着基于token认证机制的应用不需要去考虑用户在哪一台服务器登录了，这就为应用的扩展提供了便利。
- token之一，jwt
- <font color="red">token的安全在于鉴权验证，一些复杂的token机制中，一个token令牌可以包含地理位置信息，网络信息，客户端属性，包括浏览器指纹。即便token泄漏，其他人拿着这个token也难以请求成功。token都是有时效的，有时采用双token，一个用于登录访问，时效可能就一两天甚至几小时，一个用于获取新的token</font>
- 针对pc浏览器 推荐把token存到cookie然后使用httponly和处理csrf来不让黑客盗取token, 避免xss

## 流程上是这样的
1. 用户使用用户名密码来请求服务器
2. 服务器进行验证用户的信息
3. <font color="red">服务器通过验证发送给用户一个token</font>
4. 客户端存储token，并在每次请求时附送上这个token值, <font color="red">浏览器的存储技术 (cookie,localstorage) </font>
5. 服务端验证token值，并返回数据
- <font color="red">token是服务端生成下发的，如果只是随机字符串相当于session 如果携带信息相当于cookie, jwt也就是cookie进行加密的变种</font>
- 这个token必须要在每次请求时传递给服务端，它应该保存在请求头里. <font color="red">服务端要支持CORS(跨来源资源共享)策略，在服务端Access-Control-Allow-Origin: * </font>

## 使用
- 一般是在请求头里加入Authorization，并加上Bearer标注
![alt text](image.png)

## 应用场景
一般用于跨平台开发,以及接口开发 api key

## token缺点和改进
比较大的弊端，没法随时撤销某个已经经发的 token，也没法随时修改。
下面3个场景下会出现问题，大家在使用 JWT 做身份验证的时候务必留意。
1. 退出登录，没法做到真正的后台 logout，因为那个签发的 token 还在有效期内；
2. 用户信息调整后不能及时反映，比如用户的 name 修改了，或者用户的角色变化了，从原来的 admin 改成了普通用户，但是之前签发的 JWT token 还是生效状态，里面的信息还是修改前的；
3. 发现某个token被坏人拿到了，服务器端也没有效的手段立马将这个 token 置成无效。
### 改进
- 优化方案1: 黑名单模式
控制token是否有效我们这边采取的是<font color="red">黑名单模式</font>，比如某个用户改了密码就在redis中放个key叫user:ban:id，值是当前的时间戳，每次token来就解析出id和颁发时间，若是在redis中存的时间戳前颁发的token就一律无效 <font color="red">这样做是为了实现让同一用户在任一时间只有一个有效jwt</font>。 对比直接存储到redis减少存储资源

- 优化方案2: 双token设计

## uuid和jwt
- UUID
1. 高频Redis访问：每次请求必须查询Redis，对高并发场景压力大。
2. 无自描述性：UUID本身不携带信息，需依赖Redis存储完整数据。
3. 扩展性差：无法实现短期令牌自动过期
- JWT
1. 减少Redis访问频率, 仅需对有效且未过期的JWT查询Redis（例如检查是否在黑名单中），无效请求（如篡改、过期）直接拦截，降低Redis负载。
2. 自包含性与无状态验证, JWT的Payload可存储用户基础信息（如ID、角色），部分场景无需查询Redis即可完成鉴权（如读取用户ID生成日志）。
3. 灵活控制安全粒度
4. 短期令牌：JWT设置短过期时间（如15分钟），结合Refresh Token自动续期。
5. 主动注销：将需失效的JWT ID存入Redis黑名单，校验时检查是否命中。

# jwt
Json web token (JWT), 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准

- 特别适用于分布式站点的单点登录（SSO）场景。
  - <font color="red">cas验证应用的st （jwt token）的步骤就可以省略,使用非对称加密的情况下，cas把公钥分享给服务端</font>，对称加密的情况下，只有密钥的持有者cas才能验证。非对称加密的情况，公钥（cas给应用程序）也能验证经过cas私钥加密的jwt token
- JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息
- <font color="red">该token也可直接被用于认证，也可被加密。</font>
- **<font color="red"> 注意：secret是保存在服务器端的，jwt的签发生成也是在服务器端的，secret就是用来进行jwt的签发和jwt的验证，所以，它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了 </font>**

- 2个b站视频
## 无状态
无状态就是说服务器不保存令牌，有状态就是服务器会保存令牌。 jwt是通过密码学原理来保证其合法性的，也就是说服务器收到jwt，只要验签能通过，它就是合法的。反过来，传统的有状态令牌，则是通过判断令牌在服务器上的有无 来确定其合法性的，有则合法，无则非法

## jwt构成
第一部分我们称它为头部(header),第二部分我们称其为载荷(payload)，第三部分是签证(signature)
- base64加密(该加密是可以对称解密的)
``` json
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```
## header
jwt的头部承载两部分信息：
1. 声明类型，这里是jwt
2. 声明加密的算法 通常直接使用 HMAC SHA256
完整的头部就像下面这样的JSON：
``` json
{  'typ': 'JWT',  'alg': 'HS256'}
// 然后将头部进行base64加密(该加密是可以对称解密的),构成了第一部分
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```
## playload
载荷就是存放有效信息的地方。(尽量不放敏感信息)
- 标准中注册的声明 (建议但不强制使用), 过期时间之类
``` go
iss: jwt签发者
sub: jwt所面向的用户
aud: 接收jwt的一方
exp: jwt的过期时间，这个过期时间必须要大于签发时间
nbf: 定义在什么时间之前，该jwt都是不可用的.
iat: jwt的签发时间
jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。
```
- 公共的声明（自定义字段）
一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息
- 私有的声明（约定字段）
提供者和消费者所共同定义的声明

``` json
{"sub": "1234567890",  "name": "John Doe",  "admin": true}
// 然后将其进行base64加密，得到Jwt的第二部分。
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
```
## signature
jwt的第三部分是一个签证信息
- base64加密后的header和base64加密后的payload使用.连接组成的字符串，然后通过header中声明的加密方式进行加盐secret组合加密

声明的加密方式(header (base64后的) +"."+ payload (base64后的) , secret)
``` javascript
var encoded String = base64UrlEncode(header) + '.' + base64UrlEncode(payload);
var signature = HMACSHA256(encodedString, 'secret'); // TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```
客户端不可能伪造签名来获取数据，因改了payload里的任意东西，或者第一部分header的东西，那么在发给服务端后，服务端会进行header+payload再生成一次signature，只要这个signature和服务端生成的signature不一样，那就不通过

## 应用场景
1. 身份认证
2. 邮箱验证和密码重置 (链接含有用户信息)
3. 加密

# kratos框架 jwt中间件和验证码
- 官方包 github.com/golang-jwt/jwt/v4