---
title: "iTerm2与zsh" #标题
date: 2023-07-10T18:03:07+08:00 #创建时间
lastmod: 2023-07-10T18:03:07+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- 工具
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

# iTerm2
替代Mac自带的Terminal,专为 Mac OS 用户打造的命令行应用,自定义化的设置，比如自定义配色，自定义快捷键，方便的水平和垂直分屏功能等

# shell
- bash很多Linux发行版默认带的shell。 可定制性和可扩展性有限，自动补全功能不够强大
- zsh, 补全功能, 支持插件。
  - zsh 的默认配置极其复杂繁琐
- Oh My Zsh这个开源项目, 是一个方便你配置和使用Zsh的一个开源工具。完全兼容 bash 

# 前期准备
## 安装homebrew
![](image.png)

## 安装git wget curl
brew install wget curl

## 安装python
1. 官网安装包
2. brew install python3
![](image1.png)

## 安装zsh
mac一般自带，如果没有就安装
- cat /etc/shells
- brew install zsh

## 安装ohmyzsh工具

sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
- 有时候因为网络问题失败，旧继续试
- 可能设置了git代理
git config --global --unset http.proxy 和 git config --global --unset https.proxy

# 安装PowerLine字体
为了终端下能正确的显示fancy字符，需要安装powerline字体，这样，这些fancy字符不至于显示为乱码。 GitHub上已经有制作好的Powerline字体，可以下载了直接安装到系统：https://github.com/powerline/fonts

- pip install powerline-status --user  (这个是python的)

## PowerFonts字体库
- 安装字体库需要首先将项目git clone至本地，然后执行源码中的install.sh
新建一个文件夹，如：~/Desktop/OpenSource/
1. cd ~/Desktop/OpenSource/
2. git clone https://github.com/powerline/fonts.git --depth=1
3. cd fonts
4. ./install.sh
5. <font color="red">iTerm2>>Preferences>>Text>>Font</font>
![](image2.png)

# 安装配色方案
1. cd ~/Desktop/OpenSource
2. git clone https://github.com/altercation/solarized
3. cd solarized/iterm2-colors-solarized/
4. open .     //在打开的finder窗口中，双击Solarized Dark.itermcolors和Solarized Light.itermcolors即可安装明暗两种配色
5. iTerm2>>Preferences>>Colors>>Color Presets, 然后选择Solarized Dark或Solarized Light中任意一个

# 背景图
- Preferences -> Profiles -> Window -> Background Image
- 通过Blending调整图片背景的透明度

# 安装agnoster主题
1. cd ~/Desktop/OpenSource
2. git clone https://github.com/fcamblor/oh-my-zsh-agnoster-fcamblor.git  // 主题可以自己选
3. cd oh-my-zsh-agnoster-fcamblor/
4. ./install
5. <font color="red">修改配置文件里的主体配置，输入vi ~/.zshrc，打开zshrc配置文件，找到ZSH_THEME，将双引号中内容改为agnoster</font>, source ~/.zshrc使得修改生效
![](image3.png)

# 安装插件
- z 自身有得插件, 作用跳转历史目录
- 高亮补全插件
  1. cd ~/.oh-my-zsh/custom/plugins/
  2. git clone https://github.com/zsh-users/zsh-syntax-highlighting.git 高亮
     - 如果高亮插件无法生效，则可能是因为zsh-syntax-highlighting需要安装到目录 追加命令参数
     - git clone https://github.com/zsh-users/zsh-syntax-highlighting ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
  3. git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/plugins/zsh-autosuggestions 命令补全，安装到目录
  4. 修改配置vi ~/.zshrc, 请务必保证插件顺序，zsh-syntax-highlighting必须在最后一个。

- 插件配置项
```
plugins=(
    git
    extract // 一个功能强大的解压插件，所有类型的文件解压通过一个命令x全搞定
    z
    zsh-syntax-highlighting
    zsh-autosuggestions
)
```
## 隐藏用户名
DEFAULT_USER=$USER
## 缩短箭头内的路径
- 找到对应的主题，对应的配置项~/.oh-my-zsh/themes/xxx
```
vim ~/.oh-my-zsh/themes/agnoster.zsh-theme
// 把下面代码里的%~修改成%1d 
prompt_dir() {
prompt_segment green $CURRENT_FG '%~'
}
```
## 目录不够显眼
Preferences -> Profiles -> Text -> Text Rendering 把 Draw bold text in bright colors 前面的勾去掉：

## 安装其他软件，环境变量zsh终端也要加
增加环境变量  到zsh终端里  比如 安装的go  一般二进制包 安装在 usr/local/ bin 下

# 其他用法
## 配置命令别名
- Bash的配置文件叫做.bashrc，Zsh的配置文件，也放在用户当前目录，叫做.zshrc
- 在.zshrc中配置alias
```
alias gs="git status"
alias ga='git add'
alias grv='git remote -v'
alias gbr='git branch'
alias gpl="git pull"
alias gps="git push"
alias gl="git log"
alias gc="git commit -m"
alias gm="git merge"
```
## 命令
- 通过n个. 快速往上跳目录
- 通过d快速跳转历史目录，选择对应数字项
- 进程id补全
  - ps -aux|grep process_name 先拿到进程id，然后再 kill pid 来终止掉一个进程
  - 直接kill process_name按tab就会自动替换成进程id
- 自动补全路径，通过tab, 补全常用命令