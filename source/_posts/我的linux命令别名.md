---
title: 我的linux命令别名
date: 2019-09-03 14:01:46
tags:
id: 1567490532
---
在~/.bash_profile里面添加

解压
```
alias untar='tar -zxvf'
```

断点续传
```
alias wget='wget -c '
```

生成20位随机字符串
```
alias getpass="openssl rand -base64 20"
```

公网ip
```
alias ipe='curl ipinfo.io/ip'
```

局域网ip
```
alias ipi='ipconfig getifaddr en0'
```

清理屏幕
```
alias c='clear'
```

----------------------------------
参考：  
https://www.itcodemonkey.com/article/15364.html