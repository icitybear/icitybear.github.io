---
title: "前端发展历史" #标题
date: 2024-05-06T14:40:54+08:00 #创建时间
lastmod: 2024-05-06T14:40:54+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- 前端
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

- 2006 
  - 切图仔 ps dreamweaver  （css3前 图片拼接实现效果）
  - jquery对原生js封装, jQuery是一个JavaScript库，用于简化前端开发中DOM操作、事件处理、动画效果等常见任务。提供简便的api, 一份部分交互 ajax
  - <font color="red">div+css+jquery +php （jsp .net php ruby）</font>
  - sass是css预处理器

- 2009 
  - es5 改动小
  - <font color="red">node.js基于Chrome V8引擎的JavaScript运行环境</font>，可以让JavaScript代码在服务器端运行, 实现web服务(服务端)
  - 模块规范化 AMD CMD 前端工程化
  - less受到sass启发 提供高效的样式表编程

- 2010移动端 
  - 响应式 同一套api <font color="red">于是有了前后端分离</font>
  - jquery无法满足这些复杂内容:路由分发等, <font color="red">angular.js（google开发  引入双向数据绑定 依赖注入 模块化）, JavaScript框架，用于构建Web应用程序（客户端）。</font>
  - express（node.js的web服务端框架）
  - <font color="red">npm为了nodejs方便共享管理包和依赖项，完成工程化</font>

  
- 2011 bootstrap，推特开发 宫格响应式框架, 前端开发框架 提供预制丰富样式和组件（启发了后期各种ui框架）

- 2012 
  - <font color="red">TS TypeScript使js动态语言可以像静态语言开发大型项目</font>
  - webpack简单模块打包工具
  
- 2013 
  - <font color="red">react.js 用于构建用户界面的JavaScript库(fackbook开发), 客户端</font>, 单应用页面应用程序（spa）交互，虚拟dom，单向数据流 组件化开发 jsx语法 （原子）组成前端世界。 React.js并不是一个完整的Web框架，而是专注于视图层的构建，可以与各种不同的后端技术和框架结合使用。React.js的核心思想是通过组件化的方式构建用户界面，使得开发者能够更容易地管理和维护复杂的前端应用。
  - 构建工具gulp  
  - koa.js （node.js的web框架，进一步封装和优化）是express的下一代
  - nest.js（基于node.js的服务端框架）
  - github发布 electron  （chromium node.js 跨平台桌面应用程序开发）

- 2014 
  - <font color="red">vue.js结合angular.js和react.js，遍于入门，快速构建web客户端应用</font>
  - html5 canvas 多媒体 
  - css3得到主流浏览器支持

- 2015 
  - es6 <font color="red">EMCAScript6 对js进行了语法改进</font>, 类 继承 promise特性 指定了模块化标准
  - <font color="red">react native框架</font>, 使用js和react.js语法 构建原生移动应用 ios/android同一套代码
  - graphQL api查询资源

- 2016 
  - webpack支持es module
  - <font color="red">客户端：react.js和vue.js两大阵营</font>
    - next.js基于react.js的前端框架,，主要用于构建用户界面和客户端应用程序
    - nuxt.js 基于vue.js  提供开箱即用的工具, 开发者快速构建Vue.js应用程序。Nuxt.js提供了许多开箱即用的特性，例如服务器端渲染、路由、状态管理和静态文件生成等，使得开发者可以更轻松地构建复杂的客户端应用和静态网站。

- 2017 
  - 组件化开发方式 复用dom，css名存实亡 <font color="red">tailwind快速构建css样式，不需要自定义编写css</font>
  - next的默认组件tailwind （sass less完成了使命推出舞台）
  - <font color="red">Flutter google开发，构建跨平台应用框架</font>
  - <font color="red">nest.js （借鉴angular.js这个前端框架） ts写的 sprintboot 基于express.js(node.js) 服务端应用程序</font>
  
- 2018 
  - 小程序 
  - <font color="red">Uniapp跨平台应用框架 基于vue 一套代码</font>
  - Taro 基于react.js 类似uniapp

2020 
- <font color="red">vue3 组合式开发模式，支持TS</font>  OPTIONS API  COMPOSITION API
- 新的<font color="red">前端构建化工具Vite</font> 模块热更新 按需编译 快速 轻量  填补webpack

2022 华为ArkTs基于Ts扩展的语言 鸿蒙开发（其实Taro也支持）

