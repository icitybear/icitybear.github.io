---
title: "ai使用技巧" #标题
date: 2026-04-23T14:33:44+08:00 #创建时间
lastmod: 2026-04-23T14:33:44+08:00 #更新时间
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
1. rtk RTK是AI编程的Token节流神器。一个Rust二进制，零依赖，60-90%Token节省
https://mp.weixin.qq.com/s/5T-i2Vbpgm-5QVzcItju0w
``` markdown
brew install rtk
rtk init -g 

{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "rtk hook claude"
          }
        ]
      }
    ]
  }
}
``` 

2. AIClient-2-API代理

3.  init配置 权限配置
   - 起别名的情况
``` json
{
  "env": {
    "CLAUDE_CODE_NEW_INIT": 1 //新版本init支持
  },
  "permissions": {
    "allow": [],
    "deny": [],
    "defaultMode": "bypassPermissions" // 指定模型
  },
  "model": "claude-opus-4-6",
  "skipDangerousModePermissionPrompt": true, // 跳过权限
  "language": "chinese" // 语言配置
}
```

![alt text](image1.png)

- 命令行启动时跳过权限 起别名
- init   
   1. 规范CLAUDE.md @符号引用外部文件 （比如代码规范） 
   2. skills hooks对应目录

4. lsp （一般idea自带）
5. marketing  下载对应plugin 配置下mcp浏览器 卸载移除
6. /btw 问题
7. /clear 清理上下文
8. shift+tab 计划模式 子代理直接阅读项目代码

![alt text](image2.png) 对应的会话hash 可以重新打开继承上下文

9. mcp 
![alt text](image3.png)
10. 技能安装和移除
