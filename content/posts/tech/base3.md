---
title: "cdn缓存" #标题
date: 2023-10-25T14:55:28+08:00 #创建时间
lastmod: 2023-10-25T14:55:28+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- 基础
- 缓存
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

[以七牛为例](https://www.bilibili.com/video/BV1nY411m7Ap)
app  源站 多个cdn

# cdn的正确用法
1. app上传资源图片到源站
2. cdn会配置源站  如果拼接cdn地址的资源 404 就会到源站取资源 <font color="red">（回源操作）</font>
3. 所以图片违规未审核的情况下 图片链接应该是要源站的，<font color="red">只有审核通过后才能用cdn 不扩散到cdn  否则cdn域名被封</font>
   - -src 是给后台用的。是原始资源，没有CDN。给客户端的不用加-src
  比如他趣https://forum01-src.jiaoliuqu.com/516808488010008543.jpg 线上也没走cdn 因为未使用cdn的
  https://forum01.jiaoliuqu.com/516808488010008543.jpg 走cdn的情况
   - 还有比如语言违规的资源，就直接把文件移走
4. 七牛提供移动cdn文件 刷新cdn地址的，或者禁用。**本质都是加密 比如移到私有空间然后需要token才能访问。token的有效期15天。。需要访问时先获取token再拼接到url**


# 缓存是什么
![Alt text](image.png)
**客户端浏览器先检查是否有本地缓存是否过期，如果过期，则向CDN边缘节点发起请求，CDN边缘节点会检测用户请求数据的缓存是否过期，如果没有过期，则直接响应用户请求，此时一个完成http请求结束；如果数据已经过期，那么CDN还需要向源站发出回源请求（back to the source request）,来拉取最新的数据。**

# CDN缓存
浏览器本地缓存失效后，浏览器会向CDN边缘节点发起请求。类似浏览器缓存，CDN边缘节点也存在着一套缓存机制。
## CDN缓存优缺点
CDN的分流作用不仅减少了用户的访问延时，也减少的源站的负载。但其缺点也很明显：当网站更新时，如果CDN节点上数据没有及时更新，即便用户再浏览器使用Ctrl +F5的方式使浏览器端的缓存失效，也会因为CDN边缘节点没有同步最新数据而导致用户访问异常。
## CDN缓存策略
**CDN边缘节点缓存策略**因服务商不同而不同，但一般都会遵循http标准协议，<font color="red">通过http响应头中的Cache-control: max-age的字段来设置CDN边缘节点数据缓存时间。</font>

- 当客户端向CDN节点请求数据时，CDN节点会判断缓存数据是否过期，若缓存数据并没有过期，则直接将缓存数据返回给客户端；否则，CDN节点就会向源站发出回源请求，从源站拉取最新数据，更新本地缓存，并将最新数据返回给客户端。
- CDN服务商一般会提供基于文件后缀、目录多个维度来指定CDN缓存时间，为用户提供更精细化的缓存管理。
- CDN缓存时间会对“回源率”产生直接的影响。若CDN缓存时间较短，CDN边缘节点上的数据会经常失效，导致频繁回源，增加了源站的负载，同时也增大的访问延时；若CDN缓存时间太长，会带来数据更新时间慢的问题。<font color="red">开发者需要增对特定的业务，来做特定的数据缓存时间管理</font>
5：CDN缓存刷新
- CDN边缘节点对开发者是透明的，<font color="red">相比于浏览器Ctrl+F5的强制刷新来使浏览器本地缓存失效，开发者可以通过CDN服务商提供的“刷新缓存”接口来达到清理CDN边缘节点缓存的目的。</font>这样开发者在更新数据后，可以使用“刷新缓存”功能来强制CDN节点上的数据缓存过期，保证客户端在访问时，拉取到最新的数据。

# 其他扩展
cdn是有失效机制的 所以可以通过标记cdn服务器的资源失效触发回源。用fastdfs自己建，实在不行就去弄个ftp服务

- <font color="red">个人站长的预算低，服务器不禁打，就可以用cdn+ip证书的方式隐藏源站ip</font>

七牛云 kodo+cdn 弄个展示性的网站，比如企业官网啥的，只需要花个域名钱就可以了，速度还贼78快！现在我托管网站的时候，如果碰到可能会被很多人访问的内容会扔到境外 cloudflare挺好的，就是有的地方访问打不开

- 七牛云的cdn流量走https是不免费的，免费10G只限于http流量。

一般来说新增图片就是唯一不重复的，不会做替换操作，所谓类似换头像的场景不会给你一个固定图片URL而是把新的图片URL设置成头像

- 通过请求参数加版本号，需要cdn支持并开启全路径缓存 ，业务上都算常见（看后端怎么写）。缓存都是按url区别的，可以带参数也可以不带参数，还可以只带个别参数。
站在 CDN厂商立场来说，CDN管不了客户的代码怎么写，即使url带时间戳，CDN缓存的时候也可以选择忽略掉或保留,<font color="red">举例子一个图片url可能同时携带尺寸参数和鉴权参数，缓存的时候通常会保留尺寸参数但忽略鉴权参数。</font>

# 如何查看一个URL是否命中CDN缓存

- 设置cdn不缓存
header('Cache-Control: no-cache, private, max-age=0');
header('Expires: Mon, 26 Jul 2013 05:00:00 GMT');

- 主要查看响应头信息中的“X-Cache”字段。显示“MISS”，说明没有命中CDN缓存，是回源的。显示“HIT”，是命中了CDN缓存。
![alt text](image1.png)