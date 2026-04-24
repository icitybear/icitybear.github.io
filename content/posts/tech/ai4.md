---
title: "ClaudeCode" #标题
date: 2026-04-23T14:07:41+08:00 #创建时间
lastmod: 2026-04-23T14:07:41+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- ai
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

# 安装 node.js
官网选取 nvm先安装node.js  

# 安装claude code
curl -fsSL https://claude.ai/install.sh -o install.sh
- 安装失败
cat install.sh 看到 <!DOCTYPE html> 或者类似的 HTML 内容，而不是 #!/bin/bash。既然直接 curl 下不到正确的脚本
![alt text](image1.png)

- Claude Code 本质上就是个 npm 包

npm install -g @anthropic-ai/claude-code

![alt text](image2.png)

# 配置模型key
可以先用国内的模型
- 用户目录配置对应的settings.json

# AIClient-2-API代理
https://github.com/justlovemaki/AIClient-2-API 

一个强大的代理，可以统一各种仅客户端大型模型 API（如 Gemini CLI、Antigravity、Codex、Grok、Kiro 等）的请求，模拟请求，并将其封装到本地兼容的 OpenAI 接口中

## 下载镜像
``` 
# 通过国内鏡像站下载 docker 镜像
docker pull docker.1ms.run/justlikemaki/aiclient-2-api:latest

# 重命名镜像，避免后续要输入很长的镜像名 
docker tag docker.1ms.run/justlikemaki/aiclient-2-api:latest justlikemaki/aiclient-2-api:latest

# 创建配置文件日录，避免服务重启后配置丢失 
mkdir -p ~/.aiclient2api

# 启动 AICljent-2-API 服务 
docker run -d \ # 以 daemo 模式运行 
-p 127.0.0.1:3000:3000 \ # apissl 
--restart=always \ # docker 重启后自动运行服务 
-v "SHOME/.aiclient2api:/app/configs" \ # 配置日录映射 
--name aiclient2api \ # 服务命名 
justlikemaki/aiclient-2-api
``` 
## 启动容器
``` 
docker run -d -p 127.0.0.1:3000:3000 --restart=always -v "$HOME/.aiclient2api:/app/configs" --name aiclient2api justlikemaki/aiclient-2-api
``` 

## webUi登录

Docker 服务运行后，访问 http://127.0.0.1:3000 默认密码为 “admin123"，可以在web 界面里修改（"配置管理"->"高级配置"->"后台登录密码"）

![alt text](image3.png)

## 配置kiro提供商（转发kiro的）
### 查看  clientId 和 clientSecret
在进行后续操作前，确保你已经登陆过 kiro ide 或 kiro cli，确认以下文件是否存在，不存在需要先登陆kiro ide 或 kiro cli.
``` 
$ ls -l ~/.aws/sso/cache
``` 
- kiro-auth-token.json 文件
- 7xxxx.json 是文件名为40位 hash 值的json文件，里面有 clientId 和 clientSecret 字段。
![alt text](image4.png)

### 提供商池管理
在 http://127.0.0.1:3000 web界面里 

- 进入"提供商池管理"->"Claude Kiro OAuth"->"生成授权"->"导入AWS账号"->"JSON粘贴"

粘贴上面的 kiro-auth-token.json 文件里的内容，然后再将7xxxx.json 文件里的 clientId 和 clientSecret 字段添加到ison里，最终的json应该是这样的
``` json
{
  "accessToken": "auth里的xxx",
  "refreshToken": "auth里的xxx",
  "expiresAt": "2026-04-02T11:21:10.783Z",
  "clientIdHash": "hash的json文件名7xxxx",
  "authMethod": "IdC",
  "provider": "Enterprise",
  "region": "us-east-1",
  "clientId": "7xxxx的clientId",
  "clientSecret": "7xxxx的clientSecret",
}
``` 
### 验证成功
``` 
$ curl http://127.0.0.1:3000/claude-kiro-oauth/v1/messages \
-H "Content-Type:application/json" \
-H "X-API-Key:123456" \
-d '{
"model": "claude-opus-4-6",
"max_tokens": 1000,
"messages": [{"role": "user", "content": "Hello!"}]
}'
``` 
- 使用对应模型 claude-opus-4-6  claude-sonnet-4-5 
- X-API-Key 是配置的 api key，默认为"123456"，可在http://27.0.0.1:3000的“配置管理"->"基础设置"- >'API密钥“进行修改，修改后记得点击最底下的“保存配置"
![alt text](image5.png)

# Claude Code Switch ccs 
- https://github.com/kaitranntt/ccs
![alt text](image6.png)

## 安装
```
npm install -g @kaitranntt/ccs
ccs config 启动管理界面
```

## 配置kiro
![alt text](image7.png)

![alt text](image8.png)

## 启动cc
ccs kiro 启动claude code

## 解除权限授权
claude code默认运行命令，需要在终端不断进行授权，使用
```
ccs kiro --dangerously-skip-permissions
```
默认就不需要再进行授权，可以让AI把这个命令做成一个alias别名，方便启动