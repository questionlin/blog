---
title: 用javascript做爬虫的正确姿势
date: 2019-03-01 14:53:55
tags: 
- javascript
- 最佳实践
id: 1551423297
---
之前写过[用PHP做爬虫的正确姿势](/posts/1538110935/)和[用python做爬虫的正确姿势](/posts/1529320519/)。但是从上一篇文章[php异步编程](/posts/1547608999/)我们知道，在有大量的网络请求等待的情况下，异步是提高系统并发能力的手段。爬虫由于有大量的网络请求，nodejs 天然的异步成为了做爬虫的最佳选择。这篇文章介绍相应的库。

第一个是请求库，我选择的是[axios](https://github.com/axios/axios)。

第二个是dom解析库[cheerio](https://github.com/cheeriojs/cheerio)。这个基本就是 jQuery 的翻版。由于 jQuery 本身也是 javascript 实现的，所以 cheerio 比之前的 PHP 和 python 版本实现的更好。

用 javascript 做爬虫还有一个好处，就是如果页面里面有 json，不需要分割出 json，只需要给整个<script></script>段执行 eval()，然后返回想要的变量即可。当然这时候要注意改变 this 的作用域，避免污染全局变量。

--------------------------------
篇外废话：现在的后端架构基本上是一个比较简单的语言来实现应用层，加一个速度快的语言来实现数据层。javascript 因为跟 php 差不多简单，再加上本身就是异步，实现了高效的io，是比 php 更好的应用层选择。