---
title: 在Mac使用vscode开发Java8
date: 2021-02-07 10:20:20
tags: java
id: 1612664463
---
如果你可以购买 Intelli IDAE，可以关闭浏览器标签了。

# java 环境
vscode 的 java 插件只支持 java11 以上，所以要折腾一番才能开始开发。首先要学会清理现场，删除java
```sh
sudo rm -rf /Library/Internet\ Plug-Ins/JavaAppletPlugin.plugin
sudo rm -rf /Library/PreferencesPanes/JavaControlPanel.prefPane
sudo rm -rf ~/Library/Application\ Support/Java

# 先使用ls /Library/Java/JavaVirtualMachines/查询jdk名称，复制jdk名称后执行下面的命令
sudo rm -rf /Library/Java/JavaVirtualMachines/[jdk name]
```

在 mac 使用 brew 下载最新版 adoptopenjdk 和8
```sh
$ brew install adoptopenjdk
$ brew install adoptopenjdk8
```
它们都会被安装到 /Library/Java/JavaVirtualMachines/ 这个目录下

编辑 ~/.zshrc 选择默认 java 版本，注释掉就是最新版 java
```sh
export JAVA_HOME=$(/usr/libexec/java_home -v 1.8)
```

# vscode
在 vscode 编辑器首页，点 java 会自动安装扩展，或者在扩展页搜索 java Extension Pack 这个扩展。然后点击 Code -> 首选项 -> 设置，搜索 jdk，点击 java，找到 java: home，点击 在 settings.json 中编辑，加入下面配置
```json
"java.home": "/Library/Java/JavaVirtualMachines/adoptopenjdk-15.jdk/Contents/Home",
"java.configuration.runtimes": [
    {
      "name": "JavaSE-1.8",
      "path": "/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home",
      "default": true
    },
    {
      "name": "JavaSE-15",
      "path": "/Library/Java/JavaVirtualMachines/adoptopenjdk-15.jdk/Contents/Home",
    },
]
```

---------------------------------
参考：  
https://my.oschina.net/u/160225/blog/4672560