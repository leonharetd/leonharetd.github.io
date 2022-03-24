---
title: Hexo博客搭建——Hello Hexo
date: 2022-03-24 15:52:05
categories:
  - hexo
tags:
  - hexo
---

<blockquote>
  一切从创建hello hexo开始。
</blockquote> 

## Hexo
<p>
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。
</p>

## Hexo的安装
需要环境
- [nodejs](https://nodejs.org/en/)
- [git](https://nodejs.org/en/)

安装完上面的组件后，使用npm全局安装hexo-cli命令行工具。
```python
npm install -g hexo-cli
```
<!-- more -->
如果 npm下载比较慢的话
```python
# 查看默认源
npm config get registry

# 设置新源
npm config set registry https://registry.npm.taobao.org 
```
## Hello Hexo
创建博客目录
```html
hexo init <folder name>
cd <folder name>
tree -L 2
```
可以看到如下文件结构
```html
├── _config.landscape.yml
├── _config.yml
├── db.json
├── package-lock.json
├── package.json
├── public
│   ├── 2022
│   ├── archives
│   ├── css
│   ├── fancybox
│   ├── index.html
│   └── js
├── scaffolds
│   ├── draft.md
│   ├── page.md
│   └── post.md
├── source
│   └── _posts
└── themes
    └── landscape
```
这说明项目已经初始化完成。

在当前目录下执行如下命令，启动hexo服务
```python
hexo s

>>> INFO  Validating config
>>> INFO  Start processing
>>> INFO  Hexo is running at http:/localhost:4000 
```
直接访问 http:/localhost:4000

看到首页
![](blog.jpg)

over～


