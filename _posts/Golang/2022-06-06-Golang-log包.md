---
layout:     post
title:      Golang-log包
subtitle:   log包基础用法，常见写法
date:       2022-06-06
author:     Erain
header-img: img/home-bg.jpg
categories: Golang
catalog: true
tags:
    - Golang
    - Go包
---

log学习记录

# log的基础使用

格式化输出，用法同fmt打印，主要有三种:

- Print（Print \| Printf \| Println）
- Fatal（Fatal \| Fatalf \| Fatalln）
- Panic（Panic \| Panicf \| Panicln）

支持前缀*SetPrefix*、自定义打印的信息*SetFlags*、自定义输出的目的地*SetOutput*

以下是flag标签：
```go
const (
// 控制输出日志信息的细节，不能控制输出的顺序和格式。
// 输出的日志在每一项后会有一个冒号分隔：例如2009/01/23 01:23:23.123123 /a/b/c/d.go:23: message
Ldate = 1 << iota // 日期：2009/01/23
Ltime             // 时间：01:23:23
Lmicroseconds                 // 微秒级别的时间：01:23:23.123123（用于增强Ltime位）
Llongfile                     // 文件全路径名+行号： /a/b/c/d.go:23
Lshortfile                    // 文件名+行号：d.go:23（会覆盖掉Llongfile）
LUTC                          // 使用UTC时间
LstdFlags = Ldate | Ltime // 标准logger的初始值
)
```

基础用法：
```go
func init() {
log.SetOutput(CreateFile()) // SetOutput函数用来设置标准logger的输出目的地，默认是标准错误输出
log.SetPrefix("【前缀】") // 设置前缀，最前面的
// --重要--用或来实现多功能,用&来判断
log.SetFlags(log.Llongfile | log.Lmicroseconds | log.Ldate)
}


func Test0606(t *testing.T) {
log.Print("哈哈")
//log.Fatal("哈哈")// Fatal系列函数会在写入日志信息后调用os.Exit(1)
//log.Panic("哈哈")// Panic系列函数会在写入日志信息后panic
}
```

# log的常见使用

log标准库中还提供了一个创建新logger对象的构造函数–New，支持我们创建自己的logger示例。

以下是：将log输出到log文件里。
```go
func CreateFile() *os.File {
	file, err := os.OpenFile("./logTest.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
	if err != nil {
		log.Fatalln("Faild to open error logger file:", err)
	}
	return file
}

func Test06061(t *testing.T) {
l := log.New(io.MultiWriter(CreateFile(), os.Stderr), "", log.Ldate|log.Lmicroseconds)
l.Println("Erain")
}

```