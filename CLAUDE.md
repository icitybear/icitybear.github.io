# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

基于 Hugo 的个人技术博客，使用 PaperMod 主题（通过 git submodule 引入），部署在 GitHub Pages。

站点地址：https://icitybear.github.io

## 常用命令

```bash
# 本地开发预览
hugo server -D

# 构建站点
hugo --minify

# 创建新文章（使用 archetypes/default.md 模板）
hugo new posts/tech/<slug>/index.md
```

## 部署

推送到 master 分支后，GitHub Actions（`.github/workflows/hugo.yml`）自动构建并部署到 GitHub Pages。CI 使用 Hugo v0.102.3 extended。

## 内容结构

文章按分类存放在 `content/posts/` 下，使用 Hugo Page Bundle 格式（每篇文章一个目录，内含 `index.md`）：

- `posts/tech/` — 技术文章（Go、算法、AI、设计模式等）
- `posts/read/` — 阅读笔记
- `posts/life/` — 生活记录

文章图片放在对应文章目录内，或 `static/img/` 下。

## 文章 Front Matter

参考 `archetypes/default.md`，关键字段：

- `categories` / `tags` — 分类和标签
- `weight` — 设为 1 可置顶
- `draft: false` — 默认非草稿
- `mermaid: true` — 启用 mermaid 图表支持
- `cover.image` — 封面图路径

## 主题定制

PaperMod 主题通过 `themes/PaperMod/`（git submodule）引入，**不要直接修改主题文件**。所有定制通过 `layouts/` 目录覆盖实现：

- `layouts/partials/` — 覆盖主题的 partial 模板（评论、页脚、目录等）
- `layouts/shortcodes/` — 自定义短代码（bilibili、mermaid、douban、friend、collapse 等）
- `layouts/_default/` — 覆盖默认布局模板

## 关键配置（config.yaml）

- 评论系统：twikoo（Vercel 部署）
- 搜索：基于 Fuse.js 的客户端搜索
- 代码高亮：monokai 主题
- 支持打赏功能（微信/支付宝）
- 多语言配置但实际只使用中文内容
