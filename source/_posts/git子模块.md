---
title: git子模块
date: 2018-09-25 09:58:46
tags: git
id: 1537840874
---
大多数现在语言都有自己的包管理器，比如 php 的 composer，和 nodejs 的 npm。但是如果语言没有包管理器，或者想要的库没有打包，又不想手动更新，该怎么办呢？答案是 git 子模块。下面来一步一步建一个子模块。

# 添加子模块
```sh
# 首先建一个仓库
$ mkdir ljj_test
$ cd ljj_test
$ git init

# 添加一个子模块
$ git submodule add https://github.com/questionlin/php-view.git
Cloning into '/Users/simonlin/Documents/workshop/ljj_test/php-view'...
remote: Enumerating objects: 46, done.
remote: Total 46 (delta 0), reused 0 (delta 0), pack-reused 46
Unpacking objects: 100% (46/46), done.

# 查看一下
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   .gitmodules
	new file:   php-view
```
这时目录下会多出我们的子模块 php-view 和一个 .gitmodules 文件，这个文件里保存了子模块的映射关系。**这个操作在子目录下也可行。**

# 使用子模块
## 下载子模块
```sh
# 进入到 git 仓库目录
$ cd ljj_test
$ git submodule init
# 下载文件
$ git submodule update

# 或者也可以用命令
$ git clone --recursive https://github.com/chaconinc/MainProject
```

## 切换分支
```sh
$ git config -f .gitmodules submodule.php-view.branch (branch name)
```

## 更新
```sh
$ git submodule update --remote (module name)
```
这个命令会更新所有子模块，如果只想更新特定子模块，后面加上子模块名称

## 删除子模块
删除子模块的代码，然后删除 .gitmodules 里面的相关信息。

----------------------------------
参考资料：  
https://git-scm.com/book/zh/v2/Git-工具-子模块