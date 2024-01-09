---
title: "dns与域名劫持，https流程" #标题
date: 2023-10-25T15:29:42+08:00 #创建时间
lastmod: 2023-10-25T15:29:42+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- 基础
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

# 一、常规DNS解析
DNS，域名解析系统，互联网技术的基石之一，以下是常规流程。
![Alt text](dns.png)
DNS查询，会先从本地缓存查找，如果没有或者已经过期，就从DNS服务器查询，如果客户端没有主动设置DNS服务器，一般是从服务运营商DNS服务器上查找。此时将会有不可控因素。因为如果 <font color="red">使用了运营商的LocalDNS域名服务器，那么基本都会或多或少地遇到各种域名被缓存、劫持、用户跨网访问缓慢等问题</font>。比如：客户端或者服务端网站出现其他第三方广告，就是DNS被劫持的典型表现之一。

**<font color="red">为此为了解决域名被劫持，加速问题，业界提出了 IP直连服务器的方案，流程如下。</font>**

# 二、IP直连解决方案
HTTPDNS 是指“基于HTTP的域名解析服务”（HTTP Domain Name System）。它是一种通过HTTP协议来进行域名解析的服务。相比传统的DNS解析方式，HTTPDNS具有更快的解析速度和更稳定的性能。 <font color="red">它通过直接向HTTP服务器发送域名解析请求，减少了DNS解析的时间和延迟，</font>提升了用户的网络访问体验。
![Alt text](httpdns.png)

1. 客户端先通过HTTPDNS直接解析出IP地址
   - 如果成功获取到IP地址，则替换域名地址为IP地址
   - 如果获取IP地址失败，则走运营商IP解析 （一个兜底策略，基本都会走IP直连通道）
2. 如果是IP直接，先处理客户端HTTPS鉴权
3. 发起跟业务服务器的数据请求交互

以上通过直接使用IP地址与业务服务器交互，绕过运营商的DNS解析服务，即解决了域名被劫持问题。

# 三、核心问题处理（HTTPS证书鉴权）
## HTTPS
- HTTPS (Hypertext Transfer Protocol Secure) 是基于 HTTP 的扩展，用于计算机网络的安全通信，已经在互联网得到广泛应用。 <font color="red">在 HTTPS 中，原有的 HTTP 协议会得到 TLS (安全传输层协议) 或其前辈 SSL (安全套接层) 的加密。</font>因此 HTTPS 也常指 HTTP over TLS 或 HTTP over SSL。
HTTPS主要是为了解决3个问题：数据加密、数据完整性、数据真实性。
  - 数据加密  依赖SSL层对数据进行加密
  - 数据完整、数据真实  依赖CA数字证书来实现

## HTTPS交互流程
![Alt text](https.png)

HTTPS 的整个通信过程可以分为两大阶段：**证书验证和数据传输阶段，数据传输阶段又可以分为非对称加密和对称加密两个阶段**。 具体流程按图中的序号讲解

1. 客户端请求 HTTPS 网址，然后连接到 server 的 443 端口 (HTTPS 默认端口，类似于 HTTP 的80端口)。
2. 采用 HTTPS 协议的服务器必须要有一套数字 CA (Certification Authority)证书，证书是需要申请的，并由专门的数字证书认证机构(CA)通过非常严格的审核之后颁发的电子证书 (当然了是要钱的，安全级别越高价格越贵)。颁发证书的同时会产生一个私钥和公钥。。<font color="red">私钥由服务端自己保存，不可泄漏公钥则是附带在证书的信息中，可以公开的。证书本身也附带一个证书电子签名，这个签名用来验证证书的完整性和真实性，可以防止证书被篡改。</font>
3. 服务器响应客户端请求，将证书传递给客户端，证书包含公钥和大量其他信息，比如证书颁发机构信息，公司信息和证书有效期等。Chrome 浏览器点击地址栏的锁标志再点击证书就可以看到证书详细信息。
4. 客户端解析证书并对其进行验证。如果证书不是可信机构颁布，或者证书中的域名与实际域名不一致，或者证书已经过期，就会向访问者显示一个警告，由其选择是否还要继续通信。如果证书没有问题，客户端就会从服务器证书中取出服务器的公钥A。然后客户端还会生成一个随机码 KEY，并使用公钥A将其加密。
5. **<font color="red">客户端把加密后的随机码 KEY 发送给服务器，作为后面对称加密的密钥。</font>**
6. 服务器在收到随机码 KEY 之后会使用私钥B将其解密。经过以上这些步骤，<font color="red">客户端和服务器终于建立了安全连接，完美解决了对称加密的密钥泄露问题，</font>接下来就可以用对称加密愉快地进行通信了。
7. 服务器使用密钥 (随机码 KEY)对数据进行对称加密并发送给客户端，客户端使用相同的密钥 (随机码 KEY)解密数据。
8. 双方使用对称加密愉快地传输所有数据。

在使用IP直连之后，需要客户端处理服务器回传的证书鉴权操作，否则通信将失败。经过不断踩坑之后，具体核心代码实现方案如下。

# 从http升级到https
**[<font color="red">从http升级到https</font>](https://note.youdao.com/s/CUAxsN51)**

# Alamofire 的安全策略

performDefaultEvaluation：默认的策略，只有合法证书才能通过验证。
performRevokedEvaluation：对注销证书做的一种额外设置
pinCertificates：验证指定的证书，这里边有一个参数：是否验证证书链，关于证书链的相关内容可以去查一查其他更为详细的资料，验证证书链算是比较严格的验证了。如果不验证证书链的话，只要对比指定的证书有没有和服务器信任的证书匹配项，只要有一个能匹配上，就验证通过
pinPublicKeys：这个和上边的那个差不多
disableEvaluation：该选项下，验证一直都是通过的，也就是说无条件信任
customEvaluation：自定义验证，需要返回一个布尔类型的结果

# 四、Swift Alamofire 网络框架IP直连实现方案  （客户端swift的）
定义自己的ServerTrustManager
```swift
func attemptServerTrustAuthentication(with challenge: URLAuthenticationChallenge) -> ChallengeEvaluation {
    let host = challenge.protectionSpace.host

    guard challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
            let trust = challenge.protectionSpace.serverTrust
    else {
        return (.performDefaultHandling, nil, nil)
    }

    do {
        guard let evaluator = try stateProvider?.serverTrustManager?.serverTrustEvaluator(forHost: host) else {
            return (.performDefaultHandling, nil, nil)
        }

        try evaluator.evaluate(trust, forHost: host)

        return (.useCredential, URLCredential(trust: trust), nil)
    } catch {
        return (.cancelAuthenticationChallenge, nil, error.asAFError(or: .serverTrustEvaluationFailed(reason: .customEvaluationFailed(error: error))))
    }
}

//
import Foundation
import Alamofire

class HBSNetServerTrustManager: ServerTrustManager {
    var hostName: String?
    
    override func serverTrustEvaluator(forHost host: String) throws -> ServerTrustEvaluating? {
        if (String.isValidIP(ipStr: host)) {
            let evaluator = IpTrustEvaluator()
            evaluator.hostName = self.hostName ?? host
            return evaluator
        } else {
            return DefaultTrustEvaluator()
        }
    }
}

extension String {
    static func isValidIP(ipStr: String? = nil) -> Bool {
        if let ipStr = ipStr {
            let ipArr = ipStr.components(separatedBy: ".")
            if (ipArr.count == 4) {
                for str in ipArr {
                    let ipNumber = Int(str) ?? 0
                    if !(ipNumber >= 0 && ipNumber <= 255) {
                        return false
                    }
                }
                return true
            }
        }
        return false
    }
}

//定义用来处理IP的ServerTrustEvaluating
class IpTrustEvaluator: ServerTrustEvaluating {
/// 请求域名
    var hostName: String?
    
    func evaluate(_ trust: SecTrust, forHost host: String) throws {
        if String.isValidIP(ipStr: host),let hostName = self.hostName {
            try trust.af.performValidation(forHost: hostName)
        } else {
            try trust.af.performDefaultValidation(forHost: host)
        }
    }
}
```
抓包结果

![Alt text](image.png)