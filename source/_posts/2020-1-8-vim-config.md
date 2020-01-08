---
title: vim 配置
date: 2020-01-08 14:53:57
categories: 服务器
tags:
    - vim
---

在阿里云服务器用 vim 修改配置文件时，发现没有显示行号，缩进还不是4个，而且鼠标也不能用，还有些别的问题，用起来不太方便，于是就对 vim 编辑器进行些基础的配置。

<!-- more -->

## 配置文件

```bash 
# 配置所有用户的vim配置文件
vim /etc/vimrc

# 手动加载配置
source .vimrc
```

## 配置信息

```bash
# 显示行号
set nu
# 鼠标可用
set mouse=a

# 启动时隐去援助提示
set shortmess=atI

# 语法高亮
syntax on

# 使用vim的键盘模式
set nocompatible

# 不需要备份
set nobackup

# 没有保存或文件只读时弹出确认
set confirm

# tab缩进
set tabstop=4
set shiftwidth=4
set expandtab
set smarttab

# 文件自动检测外部更改
set autoread

# c文件自动缩进
set cindent

# 自动对齐
set autoindent

# 智能缩进
set smartindent

# 高亮查找匹配
set hlsearch

# 背景色
set background=dark

# 显示匹配
set showmatch

# 显示标尺，就是在右下角显示光标位置
set ruler

# 去除vi的一致性
set nocompatible

# 允许折叠
set foldenable

#####################设置折叠#####################

# 根据语法折叠
set fdm=syntax

# 手动折叠
set fdm=manual

# 设置键盘映射，通过空格设置折叠
nnoremap <space> @=((foldclosed(line('.')<0)?'zc':'zo'))<CR>
""""""""""""""""""""""""""""""""""""""""""""""
# 不要闪烁
set novisualbell

# 启动显示状态行
set laststatus=2

# 浅色显示当前行
autocmd InsertLeave * se nocul

# 用浅色高亮当前行
autocmd InsertEnter * se cul

# 显示输入的命令
set showcmd

# 被分割窗口之间显示空白
set fillchars=vert:/

set fillchars=stl:/

set fillchars=stlnc:/
```

---



## 参考链接：

- [有哪些编程必备的 Vim 配置？ - 地球的外星人君的回答 - 知乎](https://www.zhihu.com/question/19989337/answer/168444287)