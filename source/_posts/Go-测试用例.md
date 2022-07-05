---
title: Go 测试用例
index_img: /img/go4.png
banner_img: /img/go2.png
tags:
  - Go
categories:
  - Go
  - Test
date: 2022-07-03 02:44:58
---

### Go 测试用例
#### 前置条件：

##### 1、文件名须以"_test.go"结尾

##### 2、方法名须以"Test"打头，并且形参为 (t *testing.T)

##### 举例：/hello_test.go
```go
package main

import (
	"testing"
	"fmt"
)

func TestHello(t *testing.T) {
	fmt.Println("TestHello")
}

func TestWorld(t *testing.T) {
	fmt.Println("TestWorld")
}
```

##### 测试整个文件：
```shell script
$ go test -v hello_test.go
```
```
=== RUN   TestHello
TestHello
--- PASS: TestHello (0.00s)
=== RUN   TestWorld
TestWorld
--- PASS: TestWorld (0.00s)
PASS
ok  	command-line-arguments	0.007s
```
测试单个函数：
```shell script
$ go test -v hello_test.go -test.run TestHello
```
```
=== RUN   TestHello
TestHello
--- PASS: TestHello (0.00s)
PASS
ok  	command-line-arguments	0.008s
```
