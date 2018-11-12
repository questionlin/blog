---
title: tar常用命令
date: 2018-11-07 13:43:15
tags: 常用命令
id: 1541569442
---
```sh
$ tar cf etcbak.tar etc/  # 把 etc/ 打包成一个tar
$ tar xf etcbak.tar -C ./        # 解开一个tar 到当前目录
$ tar czf etcbak.tar.gz etc/ # 把 etc/ 打包压缩一个 tar
$ tar zxf etcbak.tar.gz -C ./  # 解压一个tar 到当前目录
```

- v 显示详情。例如：cvf, zxvf
- C 指定目录。不加默认当前目录