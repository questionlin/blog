---
title: nodejs使用最佳实践
date: 2021-01-07 20:46:03
tags:
 - javascript
 - 最佳实践
id: 1610023607
---
# 选择国内镜像
```sh
$ npm i -g nrm
$ nrm ls # 列出所有镜像
$ nrm use cnpm # 使用 cnpm 镜像 
```

# 升级 nodejs
```sh
$ npm i -g n
$ n lts # 升级到最新 lts 版
$ n stable # 升级到最新 stable
$ n latest # 升级到最新版
```

# 升级 package.json 里的依赖
```sh
$ npm i -g npm-check-updates
$ ncu # 查看所有依赖的最新版
$ ncu -u # 更新 package.json 依赖版本到最新版
$ npm i
```