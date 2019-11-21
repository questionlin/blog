---
title: cordova项目使用appcenter实现代码热更新
date: 2019-11-21 11:40:24
tags: 最佳实践
id: 1574307776
---
appcenter.ms 是微软的 app 管理网站，类似国内的友盟。这篇文章介绍使用 appcenter 的 codepush 功能给 cordova 项目实现代码热更新。codepush 除了支持 cordova 外，还支持 react native 等类似的项目。

安装 appcenter
```sh
$ npm install -g appcenter-cli
```

登录
```sh
$ appcenter login
```
这里要注意，用户名既不是邮箱，也不是昵称，而是系统给你分配的用户名，在账号设置里查看。比如我的是boyquestion-163.com。

安装 cordova 插件
```sh
$ cordova plugin add cordova-plugin-code-push@latest
```

查看自己的 key
```sh
$ appcenter codepush deployment list -a {用户名}/{项目名} --displayKeys
```
因为 codepush 是被收购后合并到 appcenter 的，所以这里的 key 不是 appcenter 网页上的 key。必需要使用这个命令来查看。

编辑 config.xml，加入
```
<platform name="android">
    <preference name="CodePushDeploymentKey" value="YOUR-ANDROID-DEPLOYMENT-KEY" />
</platform>
<platform name="ios">
    <preference name="CodePushDeploymentKey" value="YOUR-IOS-DEPLOYMENT-KEY" />
</platform>
```

如果你的 config.xml 里不是
```
<access origin="*" />
```
那么还应该加入
```
<access origin="https://codepush.appcenter.ms" />
<access origin="https://codepush.blob.core.windows.net" />
<access origin="https://codepushupdates.azureedge.net" />
```

在代码合适的地方加入
```js
document.addEventListener("resume", function () {
    codePush.sync();
});
```
项目就会自动更新了

下次当你编译好新的代码后只要执行
```
appcenter codepush release-cordova -a <ownerName>/<appName>
```
就可以推送了。更详细的可以查看参考里的文档。

----------------------------------
参考：  
https://docs.microsoft.com/en-us/appcenter/distribution/codepush/cordova