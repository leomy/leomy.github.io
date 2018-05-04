---
title: 使用Vim
toc: true
date: 2018-03-27 22:09:13
categories:
- editor
tags: [Editor, Vim, Usage]
---
收集了Vim的手册、基本配置和插件的使用

## 配置
包含通用的设置

```vim
"显示行号
set number

"自动缩进
set cindent

"统一缩进为4
set softtabstop=4
set shiftwidth=4

"语法高亮
syntax on

"上一行对齐格式应用到下一行
set autoindent

"智能对齐
set smartindent

"TAB键的宽度
set tabstop=4

"搜索忽略大小写
set ignorecase

"编码设置
set enc=utf-8	"VIM内部使用编码,全名encoding
set fenc=utf-8	"VIM解析出来的当前文件编码,可能会解析错误,全名fileencoding
set fencs=ucs-bom,utf-8,cp938,gb18030,big5,latin1	"VIM解析错误则用这个来猜测，全名fileencodings


"防止粘贴格式散乱
set paste
```
## 手册
![](/images/vim-shortcutKey.jpg)

## 插件
