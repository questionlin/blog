---
title: git命令参考
date: 2018-07-02 10:47:41
tags: git
id: 1530499706
---
# 查看 commit 历史
```sh
git log
```

# 显示本地执行过的 git 命令
就像 shell 的 history 一样
```sh
git reflog
```

# 撤销某个 commit/reflog，并重新添加一个 commit/reflog
```sh
git revert <commit-id>/<reflog-id>
```
只会撤销指定 commit 的改变，不影响后面的 commit。  
如果 commit 来自另一个分支，因为添加了新的 commit，再次 merge 时，文件不会合并。

# 回滚到某个 commit/reflog，并删除后面的 commit/reflog
和revert的区别：reset命令会抹去某个commit id之后的所有commit
```sh
git reset <commit-id>/<reflog-id> #默认就是-mixed参数，回滚 commit 和 add，不回滚文件
git reset HEAD^ #回滚到上一个 commit
git reset --mixed HEAD~1 #和上一个一样，HEAD~ 后跟往回退几步
git reset --hard <commit-id>/<reflog-id> #会滚到指定 commit，文件会改变，线上紧急回滚时用
```
git reset 参数默认是 --mixed，会回滚 commit 和 add，保留文件。平时使用默认就可以了。线上运维使用 --hard 回滚代码。

git reset commit-id 和 reflog-id 的区别：  
通常我们都是在新分支上开发，然后合并(merge)到发布分支，因此在发布分支，只要使用
```sh
git reset <reflog-id>
```
就可以回滚 merge 操作，即开发分支上所有的 commit。非常适合线上回滚代码。

这里给一个快速回滚到上一条 reflog 的代码：
```sh
git reflog |awk 'NR==2{print $1}'|xargs git reset --hard
```

# 切换分支
```sh
git checkout <branch-name>
```
放弃文件修改也可以用这个命令，但如果文件名和分支名相同的话就会搞混。所以放弃文件修改统一加上 --

# 放弃文件修改，从 stash 中拿出某个文件的修改
```sh
git checkout <stash@{n}> -- <file-name>
```
这是从 stash 中拿出某个文件的修改，但平时直接用作放弃文件修改。

# 在当前分支基础上新建分支
```sh
git checkout -b <branch-name>
```

# 删除分支
```sh
git branch -d <branch-name>
```

# 合并分支
```sh
git merge <branch-name>
```

--------------------------
参考资料：  
http://www.bootcss.com/p/git-guide/
https://github.com/521xueweihan/git-tips#回到远程仓库的状态