---
title: 简单哈希id
date: 2021-01-22 11:00:04
tags: 算法
id: 1611284449
---
在这篇[「分布式系统的ID」](/posts/1553830839)里介绍了哈希ID的终极方案UUID和将数字ID转哈希ID的方案[hashid](https://hashids.org/)。这篇文章介绍简单的随机ID生成方案。

# 常见的错误方案
## 预备知识
随机数分为
- 真随机数
- 不可预测：不是真的随机，但是不能被预测，是安全的随机数生成方案
- 不规则：常见的语言重的随机数生成方案，如果 seed 和 salt 一样，就能生成一样的随机数。不安全

## random()
很多语言的随机数算法——就是上面说的第三种随机数——是伪随机的，即可以被破解。同时性能也不比硬件随机数生成算法好，因此不推荐使用。不可预测随机数可以用很多方法生成。linux 环境可以用
```sh
head /dev/urandom|cksum
```

js可以用
```js
Number.parseInt(crypto.randomBytes(1).toString('hex'),16)
```

## 错误的取模运算
```js
let chars = '0-9a-z'
let randomNum = Number.parseInt(crypto.randomBytes(1).toString('hex'),16)//1个字节
let randomCharacter = chars.charAt(randomNum % 36);
```
这个方案通过获得随机数 Math.random()，来获得随机字符 randomCharacter，循环后组成随机字符串。因为1个字节有256种可能，所以得到随机字符可能性：
- 0-35入住 0-35。
- 36-71变为 0-35。
- 72-107变为 0-35。
- 108-143变为 0-35。
- 144-179变为 0-35。
- 180-215变为 0-35。
- 216-251变为 0-35。
- 252-255变为0-3。

可以看到 0-3的可能性大于其他数字。要避免这种情况，要合理配置第一步生成的随机数和被除数。



-------------------------------------
参考：  
https://gist.github.com/joepie91/7105003c3b26e65efcea63f3db82dfba