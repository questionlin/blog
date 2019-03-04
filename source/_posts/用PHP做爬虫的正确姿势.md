---
title: 用PHP做爬虫的正确姿势
date: 2018-09-28 13:01:26
tags:
- php
- 最佳实践
id: 1538110935
---
之前写过一篇[《用python做爬虫的正确姿势》](/posts/1529320519)。但我用 python 还不是那么顺手，这篇文章介绍下 php 爬虫的最佳实践。

# 抓取
一个爬虫程序至少需要抓取和解析两个部分，抓取部分我使用的是 [guzzle](https://github.com/guzzle/guzzle)。特点也是封装了会话（session），自动更新 cookies。

```sh
composer require guzzlehttp/guzzle
```

爬虫基本上都是网络请求，传统的阻塞方案非常浪费时间和计算能力，高级玩家可以使用这个基于 swoole 的异步抓取库[saber](https://github.com/swlib/saber)

```sh
composer require swlib/saber
```

# 解析
解析我使用的是 [dom-crawler](https://github.com/symfony/dom-crawler)。语法类似 jQuery，上手比较快。在浏览器 console 里复制元素的 css 选择器或者 xpath 路径，粘贴到代码里即可。这里要注意浏览器会给选择器添加元素，例如 tbody，复制出来是 table > tbody > tr。但其实 html 里没有 tbody 这个元素。手动去掉就好了。

```sh
composer require symfony/dom-crawler
```

dom-crawler 来自 php 宝库 symfony，底层依赖他们的另一个库 [css-selector](https://github.com/symfony/css-selector)，可以将 css 选择器转换成 xpath 路径。如果你只需要找到元素，那么把 xpath 传给 [DOMXPath](https://secure.php.net/manual/en/class.domxpath.php) 或 [SimpleXMLElement](https://secure.php.net/manual/en/class.simplexmlelement.php) 这两个扩展可以更快。

以下是不能使用的 css 选择器：
- 链接状态: :link, :visited, :target
- 用户行为: :hover, :focus, :active
- UI 状态: :invalid, :indeterminate (但是， :enabled, :disabled, :checked 和 :unchecked 是可用的)
- 伪元素: (:before, :after, :first-line, :first-letter)
- 带星号的: \*:first-of-type, \*:last-of-type, \*:nth-of-type, \*:nth-last-of-type, \*:only-of-type. (带元素名是可以的 (例如 li:first-of-type) 但带 \* 的不行.

#保存
再加一个封装好的数据库接口[Medoo](https://medoo.in/)，支持多种数据库，小巧方便。
```sh
composer require catfan/medoo
```