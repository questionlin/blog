---
title: redis主从切换踩坑
tags: redis
abbrlink: 23635
date: 2018-06-01 15:35:26
---
昨天配置了 redis sentinel 的主从切换，无论如何都无法成功，网上也都不到原因。

后来观察 sentinel 是正常工作的，可是服务器之间只传递 sdown 没有传递 odown，于是用这个搜到了。原来 sentinel 有 protected mode，要在配置里添加 bind 0.0.0.0。改完了之后终于切换成功了。

这个东西弄了我有10小时吧，配置模版里提都没提，真是大坑。