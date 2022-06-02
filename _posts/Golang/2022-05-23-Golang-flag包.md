---
layout:     post
title:      Golang-flag包
subtitle:   flag包基础用法，包路径/src/flag
date:       2022-05-23
author:     Erain
header-img: img/home-bg.jpg
categories: Code-Go
catalog: true
tags:
    - Golang
    - Go包
---

# 说明
该包用于设置启动程序时需要的参数，然后系统根据传入的参数运行特定的代码
注：
- 其中代码中的 *flag.Parse()* 用于解析传入的参数
- 在main函数中貌似没有办法使用*flag.Args()*

# 基础用法--main

## 代码
```go
/**
 * @Author: Erain
 * @Date: 2022/5/23
 */
package main

import (
	"flag"
	"fmt"
)

func main() {
	DefaultDir := flag.String("Directory", "/Temp/default_dir", "Choose a Directory")
	Mode := flag.Bool("Mode", false, "Choose Mode")
	Times := flag.Int("Times", 123, "Choose Times")

	flag.Parse() // 参数解析

	fmt.Println("DefaultDir:", *DefaultDir)
	fmt.Println("Mode:", *Mode)
	fmt.Println("Times:", *Times)

}
```

- 默认运行		
```
go run main\20220523.go
```
输出：
```
DefaultDir: /Temp/default_dir		
Mode: false	
Times: 123
```
---

- 指定参数运行
```
go run main\20220523.go -Mode true
```
输出：
```
DefaultDir: /Temp/default_dir
Mode: true
Times: 123
```
---
- 查看用法	
```
go run main\20220523.go -help
```
输出：
```
Usage of C:\Users\ADMIN\AppData\Local\Temp\go-build306542866\b001\exe\20220523.exe:
  -Directory string
        Choose a Directory (default "/Temp/default_dir")
  -Mode
        Choose Mode
  -Times int
        Choose Times (default 123)
exit status 2
```


# 基础用法--test

## 代码
```go
/**
 * @Author: Erain
 * @Date: 2022/5/23
 */
var modules []string
var Times int

func init()  {
	testing.Init()// 初始化test
	flag.IntVar(&Times,"Times", 123, "Choose Times")
	flag.Parse() // 参数解析
	modules = flag.Args()
}

func Test0531(t *testing.T) {
	fmt.Println("Times:", Times)
	fmt.Println("modules:", modules)
}
```

- 指定参数运行
```
go test -v 20220531_test.go -Times 666 -args haha
```
输出：
```
=== RUN   Test0531
Times: 666
modules: [haha]
--- PASS: Test0531 (0.00s)
PASS
ok      command-line-arguments  0.178s
```

于main不同的是，go test没有help函数查看命令行参数用法，必要时配合项目文档使用。


# 问题记录
在*main*函数中使用是最好的用法，但是如果要封装，使用go test去运行也是可以的。用法于在main中有些许不同。

## 单元测试，参数不生效：flag provided but not defined: -test.v
![](/img/post/Golang/flag.png)

**问题追溯**
1. 查看flag包源码，定位到这个报错由flag包的Parse()方法调用时抛出。
2. 考虑到test本身就是个框架，把flag参数放于框架内可能会有问题，于是做了以下尝试：
   ![](/img/post/Golang/flag2.png)
   发现将自定义参数接收放于全局变量中，会生效，得出结论：在运行testing中，flag参数还未被解析

**问题解决**
问题定位是参数解析顺序有问题，于是有解决方法：控制代码依次执行：初始化test->初始化flag参数->运行test。代码如上述。

## 单元测试中，flag.Args()在flag.Parse()之前调用无效
![](/img/post/Golang/flag3.png)

20220261：至今不理解，玄学问题

## flag是支持嵌套的

并且嵌套的flag可以不一样，是独立的

终端运行 go run main.go -R 111

*main*函数
```go
func main() {
	// 项目入参
	var f tools.Flag
	flag.StringVar(&f.Mirror, "M", "uatAddress", "可选环境有：petTproAddress、petAddress，要新增需兼容")
	flag.IntVar(&f.LogicalRegionId, "L", 1092, "逻辑大区id")
	flag.IntVar(&f.ServerId, "S", 8045, "区服id")
	flag.StringVar(&f.Modules, "C", "", "模块名称，不填则为全模块，以逗号间隔")
	flag.Parse() // 参数解析

	var args = []string{"test", "-v", "test-cases"}
	if f.Mirror != "uatAddress" {
		args = append(args, "-M", f.Mirror)
	}
	if f.LogicalRegionId != 1091 {
		args = append(args, "-L", strconv.Itoa(f.LogicalRegionId))
	}
	if f.ServerId != 8045 {
		args = append(args, "-S", strconv.Itoa(f.ServerId))
	}
	if len(f.Modules) > 0 {
		args = append(args, "-C", f.Modules)
	}

	cmd := exec.Command("go", args...)
	outPut, err := cmd.CombinedOutput()
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(string(outPut))
```

*test* 函数
```go
func init() {
	testing.Init() // test初始化
	flag.StringVar(&f.Mirror, "M", "uatAddress", "可选环境有：petTproAddress、petAddress、neAddress，要新增需兼容")
	flag.IntVar(&f.LogicalRegionId, "L", 1092, "逻辑大区id")
	flag.IntVar(&f.ServerId, "S", 8045, "区服id")
	flag.StringVar(&f.Modules, "C", "", "模块名称，不填则为全模块，以逗号为间隔")
	flag.Parse() // 参数解析
}
```