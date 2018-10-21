---
title: 推荐我的结巴分词http服务
date: 2018-10-21 20:08:28
tags: 安利
id: 1540123748
---
[结巴分词](https://github.com/fxsjy/jieba)是一个非常好用的分词组件。但是只支持 python 语言。其他语言想要使用的话，只能使用第三方开发的版本。由于第三方的版本不是没有实现全部功能，就是文档不全，不然就是版本落后，非要使用的话，其实不是特别有安全感。

鉴于结巴分词的接口非常简单，常用的函数只有4个，于是我用 Flask 实现了一个分词的 http 服务，地址在 [jiebahttp](https://github.com/questionlin/jiebahttp)。提供下面4个接口：
```
/cut?sentence=&cut_all=&HMM=
/cut_for_search?sentence=&HMM=
/posseg_cut?sentence=&HMM=
/tokenize?sentence=&mode=&HMM
```
分别对应结巴的 cut, cut_for_search, posseg.cut, tokenize 4 个函数。同时支持 GET 和 POST 请求。有了 jiebahttp，其他语言就不用引入不放心的第三方库，也能方便的使用了。

题外话，Elastic Stack 和 MongoDB 的成功早就告诉我们，一个开源库想要真正好用，不能依赖第三方来实现跨语言版本，而是要自己实现远程调用服务。