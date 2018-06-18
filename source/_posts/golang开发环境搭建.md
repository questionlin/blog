---
title: golang开发环境搭建
date: 2018-06-18 14:28:40
tags: golang
id: 1529303379
---
### 安装
Mac 下安装 Go 编译器只要执行
```
brew install go
```
就可以了。各 linux 也可以使用自己的包管理器直接安装

### 工作空间
首先找一个地方放我们的工作空间，比如我选的是$HOME/Documents/gowork。这个目录的位置不能是 Go 安装目录。
```
$ mkdir $HOME/Documents/gowork
```
Go代码必须放在工作空间内。它其实就是一个目录，其中包含三个子目录：
- src 目录包含Go的源文件，它们被组织成包（每个目录都对应一个包），
- pkg 目录包含包对象，
- bin 目录包含可执行命令。

下面是一个例子：
```
bin/
    hello       # 编译好的二进制文件，可执行命令
pkg/
    darwin_amd64/
        github.com/questionlin/
            stringutil.a      # 编译好的包对象
src/
    github.com/questionilin/
        hello/
            hello.go        # 源代码
            hello_test.go   # 测试文件源代码
        stringutil/
            reverse.go      # 包源码

```

### GOPATH 环境变量
执行下面的命令
```
$ export GOPATH=$HOME/gowork
$ export PATH=$PATH:$GOPATH/bin
```

### 第一个程序
我在 github 的用户名是 questionlin，我要做的第一个程序叫 hello。执行命令
```
$ mkdir -p $GOPATH/src/github.com/questionlin/hello
$ cd $GOPATH/src/github.com/questionlin/hello
$ touch hello.go
```
写入一下代码
```
package main

import "fmt"

func main() {
	fmt.Printf("Hello, world.\n")
}
```
执行一下命令编译。这个命令在任何目录都可以执行，不需要在工作空间
```
$ go install github.com/questionlin/hello
```
如果在工作空间可以省略路径
```
$ cd $GOPATH/src/github.com/questionlin/hello
$ go install
```
现在执行看看
```
$ $GOPATH/bin/hello
Hello, world.
```

### 第一个库
首先创建目录：
```
$ mkdir $GOPATH/src/github.com/questionlin/stringutil
$ cd $GOPATH/src/github.com/questionlin/stringutil
$ touch reverse.go
```
写入一下内容
```
// stringutil 包含有用于处理字符串的工具函数。
package stringutil

// Reverse 将其实参字符串以符文为单位左右反转。
func Reverse(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```
使用 go build 来编译:
```
$ go build github.com/questionlin/stringutil
```
如果你在该包的源码目录中，只需执行：
```
$ go build
```
修改原来的 hello.go 文件，加入这个包：
```
package main

import (
	"fmt"

	"github.com/questionlin/stringutil"
)

func main() {
	fmt.Printf(stringutil.Reverse("!oG ,olleH"))
}
```
然后在编译一次 hello：
```
$ go install github.com/questionlin/hello
```
现在再运行一次
```
$ hello
Hello, Go!
```
完成以上步骤后，你的工作区间应该是像最上面介绍工作空间那章里一样


### 包名
Go源文件中的第一个语句必须是
```
package 名称
```
这里的 名称 即为导入该包时使用的默认名称。 （一个包中的所有文件都必须使用相同的 名称。）

Go的约定是包名为导入路径的最后一个元素：作为 “crypto/rot13” 导入的包应命名为 rot13。

可执行命令必须使用 package main。

### 测试
Go拥有一个轻量级的测试框架，它由 go test 命令和 testing 包构成。

你可以通过创建一个名字以 _test.go 结尾的，包含名为 TestXXX 且签名为 func (t *testing.T) 函数的文件来编写测试。 测试框架会运行每一个这样的函数；若该函数调用了像 t.Error 或 t.Fail 这样表示失败的函数，此测试即表示失败。

我们可通过创建文件 $GOPATH/src/github.com/questionlin/stringutil/reverse_test.go 来为 stringutil 添加测试，其内容如下：
```
package stringutil

import "testing"

func TestReverse(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := Reverse(c.in)
		if got != c.want {
			t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```
接着使用 go test 运行该测试：
```
$ go test github.com/questionlin/stringutil
ok  	github.com/questionlin/stringutil 0.165s
```


最后给一个墙内能用的 golang 学习教程：
https://tour.go-zh.org/list

参考：https://go-zh.org/doc/
