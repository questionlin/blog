---
title: hexo+github搭建博客最佳实践
tags:
  - hexo
  - 最佳实践
date: 2018-06-01 14:54:15
id: 1528164408
---
在考察了多种方案后最终使用 hexo 搭起了博客，这篇文章写写我是怎么使用 hexo 和 github 来搭博客的。

在官网主题那里选一个喜欢的主题，我找了一个有搜索引擎配置教学的。下载后放到 themes 文件夹。然后按照官方说明配置。

hexo 不提供 markdown 编辑器，我使用 vscode，快捷键 command + k 再按 v 后可以打开一个实时预览标签页。

我有一个特殊的需求，希望可以省略部署( hexo deploy) 这个步骤。这就要求博客源文件和生成文件放在一起。github pages 提供给用户专门建博客的项目只能放生成后的文件，这个方案否定。我的做法是：
1. 把 _config.yml 里面的 public_dir 改为 docs
2. 然后建一个普通的项目仓库，在 settings 里的 github pages Source 选为 master branch/docs folder。这里顺便把 Custom domain 改为自己的域名。
3. 在域名 dns 设置里面添加 cname 指向 question.github.io
4. 在文章顶部增加 id，值为 unix 时间戳，以优化 url
5. 图床我用的是微博相册

用到的插件：
- hexo-generator-search 用来支持搜索
- hexo-generator-feed 用来生成 feed

到此我的 hexo 博客就建好了，[项目仓库在这里](https://github.com/questionlin/blog)。比官方的方法少了一步。