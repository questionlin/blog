---
title: 谷歌Colaboraty使用介绍
date: 2019-08-16 15:01:05
tags: 在线工具
id: 1565938894
---
[谷歌Colaboraty](https://colab.research.google.com)是谷歌推出的 python 练习工具，因为提供 GPU 和可以连接谷歌硬盘，非常方便。下面介绍几个常用使用方法。

1. 运行 github 上的项目
```python
!git clone https://github.com/Puzer/stylegan-encoder.git
# 行首加上“!”就可以执行 linux 命令

# 把网盘文件复制到本地
from shutil import copyfile
copyfile('/content/drive/My Drive/source/file1', 'target/file1)

# 升级 tensorflow 版本
!pip install --upgrade tensorflow-gpu

# 降级 tensorflow
!pip uninstall tensorflow
!pip install tensorflow-gpu==1.15.3

# 执行文件
import os
os.chdir("stylegan-encoder")

!python train.py
```