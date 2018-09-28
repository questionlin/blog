---
title: 用python做爬虫的正确姿势
date: 2018-06-18 19:14:27
tags:
- python
- 最佳实践
id: 1529320519
---
已经用 python 做了不知道多少爬虫了，看看网上关于这个题材的文章都已经很老了，这篇文章介绍下我的做法。

一个爬虫程序至少需要抓取和解析两个部分，抓取我使用的是 [requests](https://github.com/requests/requests)。这个库除了封装 get, post 请求外，尤其方便的是封装了会话( session )，自动更新 cookies。对抓取需要登录的网站特别好用。

解析我使用的是 [pyquery](https://github.com/gawel/pyquery)。这个库对 jQuery 达到了很高的模仿，熟悉 jQuery 的人上手非常快。通常我都是在浏览器的 console 里复制 html 元素的 css 选择器或者 xpath 路径。这里要注意浏览器会给选择器添加元素，例如 tbody，复制出来是 table > tbody > tr。但其实 html 里没有 tbody 这个元素。手动去掉就好了。

----------------------
此外 python 还有一个重量级的爬虫库叫 scrapy。这个以前会用，现在已经很少用了。如果是大工程可以考虑。