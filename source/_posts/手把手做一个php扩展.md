---
title: 手把手做一个php扩展
date: 2018-09-24 17:51:36
tags: php
id: 1537782733
---
# 搭建环境
首先确保你本地有 php 环境。然后下载 php 源码。

```sh
# 其实只需要里面的几个文件，所以下载 zip 包也行。
$ git clone https://github.com/php/php-src.git
$ cd php-src/ext
# 用模版新建一个项目，名为 rjson
$ ./ext_skel.php --ext rjson
$ cd rjson
$ phpize
$ ./configure --with-php-config=/usr/local/bin/php-config
$ make
$ make test
$ make install
```

现在 rjson.so 已经在扩展目录下了，需要在 php.ini 最底部加上

```
extension=rjson.so
```

看看扩展加载了没有

```sh
$ php -m | grep rjson
rjson
```

新建一个文件 a.php
```php
<?php
echo rjson_test2('ljj');
```
执行看看
```sh
$ php a.php
Hello ljj
```
我们看到模版自带的测试函数已经可以执行了

# 开始编写
打开文件 rjson.c，可以找到 rjson_test1 和 rjson_test2 的定义。我们照着写一个 rjson_test3
```c
// 在下面这个数组里增加一行，第二个参数是函数参数的定义，这里省略
static const zend_function_entry rjson_functions[] = {
    PHP_FE(rjson_test3,     NULL)
}

// 最外层增加rjson_test3，可以写在 rjson_test2 后面
PHP_FUNCTION(rjson_test3)
{
	zend_string *retval;
	retval = strpprintf(0, "I'm %s", "ljj");
	RETURN_STR(retval);
}
```
然后再 make install

修改 a.php
```php
echo rjson_test3();
```
执行
```
$ php a.php
I'm ljj
```
到此，一个没有参数的函数就完成了。写 php 扩展的时候要用到很多 php 自己定义的 c 函数或者宏指令。比如 RETURN_STR() 不能返回 char*，只能返回 zend_string*。这些地方参考《PHP7内核剖析》。

-------------------------------
参考资料：  
mac 系统下开发一个PHP扩展：https://blog.csdn.net/imbibi/article/details/79354464  
用C/C++扩展你的PHP：http://www.laruence.com/2009/04/28/719.html  
《PHP7内核剖析》