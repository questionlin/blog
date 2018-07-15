---
title: 用geohash来做地理距离计算
date: 2018-07-15 23:25:58
tags:
id: 1531668376
---
在知道 geohash 之前，对于查找附近的人这种需求，我只知道用自己的经纬度和对方的经纬度经过勾股定理来计算距离，但因为这样的方法实在是太慢没法用，最后还是用了第三方的 API 来获得结果。这片文章就来介绍一下 geohash。

geohash 是对经纬度做一系列计算，最后的到一个字符串，经纬度越精确，字符串越长，而距离越近的两点，字符串左边相同的字符就越多。例如使用 redis 的 geohash 命令得到上海人民广场地铁站的 geohash 是 wtw3sqgg0v0，上海博物馆是 wtw3sqh3ng0，老西门地铁站是 wtw3ss61250。可以看到人民广场离博物馆比较近，前6位字母是一样的，而距离较远的老西门则只有前5位一样了。这个算法可以设定精确度，越精确字符串越长。

现在已经很清楚了，对于位置固定不变的东西，可以在 MySQL 保存 geohash 后用 Like 'xxx%' 来查找，缓存可以用 redis 的 geoadd 和 geohash。但如果需求是一个位置实时在变的出租车，我们不是要在缓存找到出租车的 geohash，而是要根据近似的 geohash 来找到出租车的信息。可以用 redis 的 GEORADIUSBYMEMBER 命令，将数据库 id 作为 key。

Geohash 具体算法实现这里就不写了，可以看维基百科(https://en.wikipedia.org/wiki/Geohash)。使用 geohash 虽然能快速的得到附近的物品，却不能得到距离。如果需要距离，可以取出经纬度后再计算。

最后推荐一个 [geohash 的 PHP 扩展](https://github.com/taogogo/geohash-php-extention)
