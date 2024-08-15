---
title: "web缓存" #标题
date: 2024-08-15T10:08:11+08:00 #创建时间
lastmod: 2024-08-15T10:08:11+08:00 #更新时间
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

# 第一部分 Web缓存是什么

- 场景1
测试妹子测功能时会说为什么我的浏览器的显示乱七八糟，我的界面怎么跟别人浏览器上不一致？旁边的人会提醒说：清下缓存试试

- 场景2
开发改了代码，上了环境，发现不生效，这时候首先就是清缓存，清了浏览器缓存发现还是不行，再检查，发现是反向代理缓存。
那么，当我们谈WEB缓存的时候，我们说的是什么？什么地方可以缓存？什么时候用什么缓存？使用不当会带来什么问题，我们怎么避免？
会不会傻傻分不清楚，那我们就来理一理，看看web缓存究竟是什么？

- 缓存：缓存就是把数据或者我们需要取到的内容，放到能更快访问的地方。缓存对于前端后端的coder来说，应该都不陌生，不论前端后端，我们使用缓存都是为了提升性能。

Web缓存：按照上面的逻辑，就是为了提升Web页面访问的性能，把能缓存的页面or数据缓存到能够更快取得的地方。<font color="red">广义的Web缓存也可以包括服务器缓存，本文为了与服务器缓存区分，不包含服务器缓存。</font>

# 第二部分 Web缓存的类型

在典型的web应用中，一个浏览器发起的请求，会经过下图中的几个步骤（其中CDN、反向代理是可选的），那么缓存的地方或者层次也很好理解，就是下图中的<font color="red">浏览器、反向代理、cdn。</font>


# 第三部分 浏览器缓存

以chrome浏览器为例，打开chrome开发者工具，再选择“Resources”中看到所有的缓存类型

- [http请求的缓存](https://juejin.cn/post/6844903618890465294)

![alt text](image1.png)
1. 强缓存阶段：先在本地查找该资源，如果发现该资源，并且其他限制也没有问题(比如:缓存有效时间)，就命中强缓存，返回200，直接使用强缓存，并且不会发送请求到服务器
2. 弱缓存阶段：在本地缓存中找到该资源，发送一个http请求到服务器，服务器判断这个资源没有被改动过（或者最后修改时间较旧，说明资源无新修改），则返回304，让浏览器使用该资源。
3. 缓存失败阶段(重新请求)：当服务器发现该资源被修改过，或者在本地没有找到该缓存资源，服务器则返回该资源的数据。

# 一、Frames 页面缓存
## （一）控制缓存的属性
浏览器在发送文件请求时，可以根据协议头判断从服务器端请求文件还是从本地缓存读取文件，主要判断依据是expires和etag
- <font color="red">影响浏览器的文件缓存主要有几个属性：expires(Cache-Control)、Etag、Last-Modified，这三个属性是由http协议定义的。</font>
1. Expires
用于设置静态资源的过期时间。

2. Cache-Control
Cache-Control可以用于控制是否缓存、缓存的读取权限、资源的有效期。只不过Cache-Control的选择更多，设置更细致，如果同时设置的话，其优先级高于Expires。
- （1）public 指示响应数据可以被任何客户端缓存
- （2）private 指示响应数据可以被非共享缓存所缓存。这表明响应的数据可以被发送请求的浏览器缓存，而不能被中介所缓存
- （3）no-cache 指示响应数据不能被任何接受响应的客户端所缓存
- （4）no-store 指示所传送的响应数据除了不能被缓存，也不能存入磁盘。一般用于敏感数据，以免数据被复制。
- （5）must-revalidate 指示所有的缓存都必须重新验证，在这个过程中，浏览器会发送一个If-Modified-Since头。如果服务器程序验证得出当前的响应数据为最新的数 据，那么服务器应当返回一个304 Not Modified响应给客户端，否则响应数据将再次被发送到客户端。
- （6）proxy-revalidate 与must-revalidate相似，不同的是用来指示共享缓存。
- （7）max-age:（单位秒） 数据经过max-age设置的秒数后就会失效，<font color="red">相当于HTTP/1.0中的Expires头。</font>如果在一次响应中同时设置了max-age和Expires，那么max-age将具有较高的优先级。（注：ngnix设置expires会被转换为max-age）

3. Last-Modified/If-Modified-Since
- Last-Modified：标示这个响应资源的最后修改时间。web服务器在响应请求时，告诉浏览器资源的最后修改时间。
- If-Modified-Since：当资源过期时（使用Cache-Control标识的max-age），发现资源具有Last-Modified声明，则再次向web服务器请求时带上头 If-Modified-Since，表示请求时间。
  - web服务器收到请求后发现有头If-Modified-Since 则与被请求资源的最后修改时间进行比对。若最后修改时间较新，说明资源又被改动过，则响应整片资源内容（写在响应消息包体内），HTTP 200；
  - 若最后修改时间较旧，说明资源无新修改，则响应HTTP 304 (无需包体，节省浏览)，告知浏览器继续使用所保存的cache。

4. Etag/If-None-Match
- Etag/If-None-Match也要配合Cache-Control使用。
- <font color="red">Etag：web服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识（生成规则由服务器定义）。</font> nginx中，etag会默认增加，如果需要关闭，需要在配置文件中设置：etag off;
If-None-Match：当资源过期时<font color="red">（使用Cache-Control标识的max-age）</font>，发现资源具有Etage声明，则再次向web服务器请求时带上头If-None-Match （Etag的值）。web服务器收到请求后发现有头If-None-Match 则与被请求资源的相应校验串进行比对，决定返回200或304。

## （二）用户行为与缓存
![alt text](image2.png)

## （三）如何控制缓存
1. web服务器配置
以ngnix为例，在nginx.conf中设置：

```
location~ .*\.(gif|jpg|png|htm|html|css|js|flv|ico|swf)(.*) {
    expires 1d;
}
``` 
上述配置表示这些静态文件1天后过期。如果想配置为完全不缓存，那么可以设置为expires -1；（后面的数字配置为负数），返回的header会被设置为Cache-Control:no-cache

2. 后台代码写入
``` php
# 比如php代码
/**
 * 设置cdn不缓存
 */
static public function noCache()
{
   // 设置cdn不缓存
   header('Cache-Control: no-cache, private, max-age=0');
   header('Expires: Mon, 26 Jul 2013 05:00:00 GMT');
}

/**
 * 设置缓存时间
 * @param int $time
 */
static public function setCacheTime($time = 60)
{
   $now = time();
   //must-revalidate 一定要有，ats才有效果
   header("Cache-Control: max-age=$time, must-revalidate");
   header("Expires: " . gmdate("D, d M Y H:i:s", $now + $time) . " GMT");
}
```
3. html 的meta标签
``` html
<meta http-equiv="Cache-Control" content="max-age=7200" /> 
```

## （四）缓存的问题和解决办法
1. 引入缓存之后，主要有两个问题：
- （1）浏览器不知道有资源更新，还是使用缓存中的老文件。
- （2）各个文件缓存策略不一致，有关联关系的文件，有的从服务器加载，有的直接取浏览器缓存的，这样有可能会导致界面混乱。

2. 解决方式
- （1）Etag或Last-modified
Etag是服务端根据文件信息生成的字符串，当服务端文件更新时，Etag也会变化，这样能保证当服务端文件更新时，取到新的文件内容。
但是Etag这种解决方式的问题是，请求还是会发到服务端，由服务端进行判断。
Last-modified与Etag类似。
- （2）文件名后缀
构建过程中，把构建生成的文件加上随机后缀，主入口html中的引用文件在构建中替换为增加了文件名后缀的；主入口文件配置为不缓存。
当服务端更新文件时，由于文件名后缀更改，浏览器缓存匹配不上，会直接到服务端获取，服务端没有更新文件时，在浏览器缓存获取。
这种方式效果较好，但是需要引入构建，对于已经使用了前端构建的web应用比较适用。

# 二、cookie

cookie是一种能够让网站服务器把少量数据储存到客户端的硬盘或内存，或是从客户端的硬盘读取数据的一种技术。当我们浏览某网站时，由Web服务器置于你硬盘上的一个非常小的文本文件，它可以记录用户ID、密码、浏览过的网页、停留的时间等信息。

<font color="red">cookie以键值对的方式来存储，有数量和大小的限制，数量各个浏览器不同，大小不能超过4K。</font>

## （一）设置cookie的方式：

1、浏览器

浏览器提供了操作cookie的方式，可以对cookie进行设置、读取、删除。另外,cookie也可以设置过期时间。

浏览器获取cookie的方式：

document.cookie

2、服务器

很多时候，我们会使用cookie来做协助做会话管理，登录成功后，由服务端将sessionid信息写入cookie,后续客户端发送的所有请求都携带cookie信息，服务端验证cookie中的sessionid信息，判断此请求是否合法。以java为例，服务端写入cookie的方法如下：
``` 
Cookie cookie = new Cookie("sessionid",URLEncoder.encode("fejerwiie2234","UTF-8"));
response.addCookie(cookie); 
```

## （二）cookie的属性
![alt text](image3.png)

# 三、localStorage

<font color="red">localstorage是Html5中新加入的特性，引入localstorage主要是作为浏览器本地存储，解决cookie作为存储容量不足的问题。localstorage是一种持久化的存储。
同样，localstorage也是一个key-value形式的存储。</font>

- （一）浏览器提供了localstorage的增删改查方法
``` javascript
增加/修改：window.localStorage.setItem("username","admin");
查询：window.localStorage.getItem("username");
删除：window.localStorage.removeItem("username","admin");
```
- （二）使用localstorage的注意事项
localstorage中存储的value只能是字符串型，如果要存储对象，需要转换为字符串再进行保存。

# 四、sessionStorage
sessionStorage用于本地存储一个会话（session）中的数据，<font color="red">这些数据只有在同一个会话中的页面才能访问并且当会话结束后数据也随之销毁。因此sessionStorage不是一种持久化的本地存储，仅仅是会话级别的存储。</font>
同样，浏览器也提供了sessionStorage的增删改查方法,与localStorage一致，只是获取方法为：<font color="red">window.sessionStorage</font>

# 五、IndexedDB
**IndexedDB也是html5提供的**，能够在客户端存储大量的结构化数据的数据库，并且提供API进行高效检索。IndexedDB的初始大小是50M，还可以增加，就存储量来说，秒杀其他存储方式。
但是它的缺点也很明显，IndexedDB并不是所有主流浏览器都支持，<font color="red">比如IE9、IE10和IE11都不支持</font>，所以，如果你的用户群还使用着IE系列的浏览器，IndexedDB就不用考虑了。

- Web SQL
此方案在W3C已经废弃，不再维护，替代方案是IndexedDB。
- application cache
该特性已经从 Web 标准中删除。

# 八、Cache Storage
**该方案是一个实验性的方案，并不是所有浏览器都支持。**
CacheStorage是在ServiceWorker的规范中定义的。CacheStorage 可以保存每个serverWorker申明的cache对象，cacheStorage有open、match、has、delete、keys五个核心方法，可以对cache对象的不同匹配进行不同的响应。

# 九、Services Worker
**service worker也是一个实验性的方案，并不是所有浏览器都支持。**
service worker提供了很多新的能力，使得web app拥有与native app相同的离线体验、消息推送体验。
Service worker可以：
1. 后台消息传递
2. 网络代理，转发请求，伪造响应
3. 离线缓存
4. 消息推送
   
可以参考(https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers)

# 第四部分 CDN缓存
- CDN的分流作用不仅减少了用户的访问延时，也减少了源站的负载
- 源服务器资源更新时，主动刷新CDN缓存

{{< innerlink src="posts/tech/base3.md" >}} 

# 第五部分 反向代理缓存
Web服务器隐藏在代理服务器之后，实现这种机制的服务器称作反向代理服务器(Reverse Proxy Server)。此时，Web服务器成为后端服务器，反向代理服务器称为前端服务器。
引入反向代理服务器的目的之一就是基于缓存的加速。我们可以将内容缓存在反向代理服务器上，所有缓存机制的实现仍然采用HTTP/1.1协议。
- 反向代理：通过反向代理实现负载均衡
- [【有道云笔记】Nginx Php-fpm运行原理详解](https://note.youdao.com/s/cK0UpFxl)

- [【有道云笔记】Nginx配置文件详细说明，包括基本配置，反向代理配置，expire缓存过期，适合接入CDN的配置](https://note.youdao.com/s/2yvKGENN)

## 反向代理缓存配置
``` 
以通常使用的反向代理--ngnix为例，实现缓存的配置如下：
1. proxy_cache_path
语法：proxy_cache_path path [levels=number] keys_zone=zone_name:zone_size [inactive=time] [max_size=size];  
默认值：None  
使用字段：http  
指令指定缓存的路径和一些其他参数，缓存的数据存储在文件中，并且使用代理url的哈希值作为关键字与文件名。levels参数指定缓存的子目录数，例如：
proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=one:10m;
文件名类似于：
/data/nginx/cache/c/29/b7f54b2df7773722d382f4809d65029c
levels指定目录结构，可以使用任意的1位或2位数字作为目录结构，如 X, X:X,或X:X:X 例如: “2”, “2:2”, “1:1:2“，但是最多只能是三级目录。
  
2. proxy_cache
语法：proxy_cache zone_name;  
默认值：None  
使用字段：http, server, location  
设置一个缓存区域的名称，一个相同的区域可以在不同的地方使用。  

3. proxy_cache_valid
语法：proxy_cache_valid reply_code [reply_code …] time;  
默认值：None  
使用字段：http, server, location  
为不同的应答设置不同的缓存时间，例如：
proxy_cache_valid 200 302 10m;
proxy_cache_valid 404 1m;
为应答代码为200和302的设置缓存时间为10分钟，404代码缓存1分钟。  
如果只定义时间：
proxy_cache_valid 5m;
那么只对代码为200, 301和302的应答进行缓存。  
同样可以使用any参数任何应答。
proxy_cache_valid 200 302 10m;
proxy_cache_valid 301 1h;
proxy_cache_valid any 1m;
``` 