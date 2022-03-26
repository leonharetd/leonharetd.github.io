---
title: Hexo 使用简介
date: 2022-03-26 15:28:09
categories:
  - hexo
tags:
  - hexo
---

<blockquote>
  介绍hexo的使用。
</blockquote> 

### 创建一篇文章
hexo的原始文章都存放在source文件夹下，可以自己创建md文件放入该目录，也可以通过hexo提供的命令行自动生成。
```shell
$ hexo new [layout] <title>
```
layout为hexo的文件类型，默认支持如下三种类型
|  layout   | 存放路径  | 说明|
|  :----:  | :----:  | :---: |
| post  | source/_posts | 文章 |
| page  | source | 页面 |
| draft	| source/_draft | 草稿|
```css
$ hexo generate
$ hexo server
```
访问http://localhost:4000/ 就能看到新生成的文章。
<!-- more -->
draft用来生成文章草稿，保存在source/_draft下，可使用
```shell
$ hexo publish [layout] <title>
```
将draft文章发布至source/_post下。

### 创建一篇带资源的文章
hexo是用md文件生成html文件，对包含静态资源链接需要特殊解析，不然发布后资源无法加载，v3.0之前需要用插件，还有各种书写语法支持才能实现转换。现在v3.1以后hexo推出了自己的资源文件夹来解决这个问题。用户需要开启设置，然后就能像写正常markdown一样了。
#### 开启资源文件配置
项目目录的_config.yaml增加如下配置。
```yaml
post_asset_folder: true
marked:
  prependRoot: true
  postAsset: true
```
#### 生成资源文件
我们注意到在new post的时候生成了两个东西，一个是文件，另一个是目录，这个目录就是静态资源文件，这篇文章的所有静态资源都可以放在这个文件里，hexo会自动引用。
#### 静态资源的使用
直接在文章中使用正常的markdown语法就可以了。
<code>
\!\[\]\(image.jpg\) 就可以被解析为正确的 \<img src="xxxxxx"\>.
</code>

### layout模版管理
#### 模版生成
layout模版保存在Scaffold文件里，在新建文章时，hexo会根据文件夹对应的内容来生成文件。
```shell
$ hexo new photo "My Gallery"
```
在执行这行指令时，Hexo 会尝试在 scaffolds 文件夹中寻找 photo.md，并根据其内容建立文章。
#### 模版变量
模版文件最上方以 --- 分隔的区域，用于指定个别文件的变量
```yaml
title: Hexo 使用简介
date: 2022-03-26 15:28:09
categories:
  - hexo
tags:
  - hexo
```


目前支持一下属性：
|属性名| 说明| 默认值|
| :---: | :---: | :---: |
| title	|标题	|文章的文件名|
| date	|建立日期	|文件建立日期 |
| updated |	更新日期 |	文件更新日期 |
| comments |	开启文章的评论功能 |	true|
| tags |	标签 |（不适用于分页）	|
| categories |	分类 |（不适用于分页）	|
| permalink	| 覆盖文章网址|

### Hexo常用命令行
#### 生成博客目录
```shell
$ hexo init [folder]
```
#### 生成博客文章
```shell
$ hexo new [layout] <title>
```
#### 根据markdown生成静态文件，增量生成。
```shell
$ hexo generate
```
#### 发布草稿
```shell
$ hexo publish [layout] <filename>
```
#### 启动hexo服务
```shell
$ hexo server
```
#### 清除所有生成的静态文件，一般用于重新生成时使用。
```shell
$ hexo clean
```
#### debug模式，展示更多的server日志，便于排查发布时的问题。
```shell
$ hexo --debug
```

