---
title: "加密解密，签名和验签" #标题
date: 2023-10-26T15:05:09+08:00 #创建时间
lastmod: 2023-10-26T15:05:09+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- 基础
- 网络
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
# 概念
1. 加密和解密
   - 发送方利用接收方的公钥对要发送的明文进行加密,接受方利用自己的私钥进行解密,其中公钥和私钥匙相对的,任何一个作为公钥,则另一个 就为私钥.**但是因为非对称加密技术的速度比较慢,所以,一般采用对称加密技术加密明文,然后用非对称加密技术加密对称密钥,即数字信封技术.**

2. 签名和验证
   - 发送方用特殊的hash算法，由明文中产生固定长度的摘要，然后利用 自己的私钥对形成的摘要进行加密，这个过程就叫签名。接受方利用 发送方的公钥解密被加密的摘要得到结果A，然后对明文也进行hash操作产生摘要B.最后,把A和B作比较。此方式既可以保证发送方的身份不 可抵赖，又可以保证数据在传输过程中不会被篡改。

# 加密分类
1. 单向加密：经过对数据进行摘要计算生成密文，密文不可逆推还原。算法表明：Base64，MD5，SHA;
2. 双向加密：与单向加密相反，能够把密文逆推还原成明文，双向加密又分为对称加密和非对称加密。
   1. 对称加密：指数据使用者必须拥有相同的密钥才能够进行加密解密，就像彼此约定的一串暗号。算法表明：DES，3DES，AES，IDEA，RC4，RC5;
   2. 非对称加密：相对对称加密而言，无需拥有同一组密钥，非对称加密是一种“信息公开的密钥交换协议”。非对称加密须要公开密钥和私有密钥两组密钥，公开密钥和私有密钥是配对起来的，也就是说使用公开密钥进行数据加密，只有对应的私有密钥才能解密。这两个密钥是数学相关，用某用户密钥加密后的密文，只能使用该用户的加密密钥才能解密。若是知道了其中一个，并不能计算出另一个。所以若是公开了一对密钥中的一个，并不会危害到另一个密钥性质。这里把公开的密钥为公钥，不公开的密钥为私钥。算法表明：RSA，DSA。
   - [为什么用公钥加密却不能用公钥解密？](https://mp.weixin.qq.com/s/v5mDukjQbtnyY62zEyFwtQ)
  
# 数字信封技术例子
{{< innerlink src="posts/tech/base4.md" >}}  
【有道云笔记】HTTP 升级到 HTTPS 基础知识详解 - CSDN博客 https://note.youdao.com/s/CUAxsN51

- [【有道云笔记】常见加密 ](https://note.youdao.com/s/ECnnRMTX)
- [【有道云笔记】AES,SHA1,DES,RSA,MD5区别](https://note.youdao.com/s/Nh8om4iN)

# PHP通用的 OPENSSL 方式实现RSA算法，DES算法 
PHP通用的 OPENSSL 方式实现RSA算法，DES算法 https://note.youdao.com/s/LUi62bpr

# SM国产商密算法
https://git.internal.taqu.cn/go/g3