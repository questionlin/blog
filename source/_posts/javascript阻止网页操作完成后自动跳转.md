---
title: javascript阻止网页操作完成后自动跳转
tags: javascript
date: 2018-06-01 15:41:16
id: 1528164457
---
阻止跳转的代码：
```js
$(window).on('beforeunload', () => { console.log('leave'); return false; });
```
```js
window.addEventListener('beforeunload', function(){console.log('leave');return false;});
```

参考来源：https://jingyan.baidu.com/article/574c52190bc5ac6c8d9dc196.html