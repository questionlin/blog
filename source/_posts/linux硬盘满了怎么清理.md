---
title: linux硬盘满了怎么清理
date: 2018-06-11 10:52:25
tags: 运维
id: 1528685621
---
1. 使用 df -h 查看硬盘剩余空间
2. 使用 du --max-depth=1 -h 一级一级查看硬盘文件占用，找到适合删掉的文件
3. 如果是只想删掉大文件，可以使用 find . -maxdepth 1 -size +100M
```sh
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        30G   12G   17G  40% /
devtmpfs        7.8G     0  7.8G   0% /dev
/dev/vdb         99G   55G   39G  59% /opt


$ du --max-depth=1 -h
272K ./.gconf 
32K ./.mcop 
16K ./.redhat 
1.7M ./.thumbnails 
8.0K ./.gconfd 
7.5M . 

$ find . -maxdepth 1 -size +100M
./.gconf 
./.mcop 
./.redhat 
./.thumbnails 
./.gconfd 
```