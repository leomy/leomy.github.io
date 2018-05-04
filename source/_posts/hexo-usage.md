---
title: 使用Hexo的搭建博客
toc: true
date: 2018-03-24 16:01:55
categories: 
- hexo
tags: [Hexo, Usage]
---

Hexo使用`Markdown`解析文章，其<a href="https://hexo.io/">官网</a>提供许多<a href="https://hexo.io/plugins/">插件</a>、<a href="https://hexo.io/themes/">主题</a>来生成静态页面。下面的使用参考<a href="https://hexo.io/zh-cn/docs/">官方中文文档</a>
## 安装
需要提前安装好`Node.js`、`Git`，
```sh
npm install -g hexo-cli     # 安装hexo-cli命令行
hexo init blog-with-hexo    # 初始化一个hexo项目
cd ./blog-with-hexo         # 进入hexo项目
npm install                 # 安装相应的模块

##
## 下面的根据实际情况
##
npm install hexo-deployer-git --save        # 安装git部署模块
npm install hexo-generator-search --save    # 安装搜索模块
```
## 主题
有许多<a href="https://hexo.io/themes/">官方主题</a>，下面介绍一款主题 -- <a href="https://github.com/tufu9441/maupassant-hexo">maupassant-hexo</a>
要使用主题必须在全局的`_config.yml`中启用主题
- 下载 
    ```sh
    git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant
    npm install hexo-renderer-pug --save
    npm install hexo-renderer-sass --save
    ```
- 配置
    - 在全局`_config.yml`的配置国际化 `language: zh-CN`
    - 启用站内搜索
        - 在全局`_config.yml`的配置
            ```yml
            search:
                path: search.xml
                field: post
            ```
        - 在`themes/maupassant/_config.yml`中配置
            ```yml
            google_search: false 
            baidu_search: false
            self_search: true
            ```
    - 菜单的`about`
        使用`hexo new page about` 并生成`source/about/index.md`，编辑该文档