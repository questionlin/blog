---
title: 手机连接电脑用chrome调试网页
date: 2019-12-22 10:58:56
tags: 最佳实践
id: 1576983574
---
# 安卓
## 安卓打开开发者模式
小米手机需要在 设置=》我的设备全部参数=》MIUI版本 点5次，打开开发者模式

然后在开发者模式里打开 USB 调试

***然后连接电脑的时候会弹出 USB 的用途，选择传输照片（PTP）模式***

## 电脑端
windows 需要安装驱动

chrome 打开 chrome://inspect 就能看到设备了

## cordova
cordova 调试正式版 app，需要在 AndroidManifest.xml 的 <application> 节点里加入
```
android:debuggable="true"
```

参考[https://cordova.apache.org/docs/en/latest/guide/next/#debugging-cordova-apps](https://cordova.apache.org/docs/en/latest/guide/next/#debugging-cordova-apps)