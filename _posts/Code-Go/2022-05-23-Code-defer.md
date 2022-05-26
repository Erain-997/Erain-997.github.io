---
layout:     post
title:      Code-defer
subtitle:   defer的基础用法，defer面试题的坑，defer原理
date:       2022-05-26
author:     Erain
header-img: img/home-bg.jpg
categories: Code-Go
catalog: true
tags:
    - Golang
    - Code
---

## 第一题
```go
func f1() (r int) {
	// 1.赋值
	r = 1
	// 2.r 作为函数参数，不会修改要返回的那个 r 值
	defer func(r int) {
		r = r + 5
	}(r)
	// 3.空的 return
	return
}
```
运行结果：1	

知识点：
1. 在局部作用域中，命名的返回值内同名的局部变量屏蔽
2. defer函数的参数传递为值传递时，不影响主程序数值
3. 	return 这条语句并不是一个原子指令，经过编译之后，变成了三条指令：
	1. 返回值 = xxx
	2. 调用 defer 函数
	3. 空的 return

## 第二题
```go
func Test04826(t *testing.T) {
	f, err := os.Open("file")
	//defer f.Close()// 错误defer的位置，因为这里一旦没有的话，f是个nil
	if err != nil {
		return
	}
	defer f.Close() // 正确defer的位置

	b, err := ioutil.ReadAll(f)
	println(string(b))
}
```
知识点：defer用于关闭文件、关闭数据库等场景时，要确认开启/连接了，才关闭，否则会panic

## 第三题
```go
type Slice []int

func NewSlice() Slice {
	return make(Slice, 0)
}
func (s *Slice) Add(elem int) *Slice {
	*s = append(*s, elem)
	fmt.Println(elem)
	return s
}

func Test04282(t *testing.T) {
	s := NewSlice()
	// 1.Add() 方法的返回值依然是指针类型 *Slice，所以可以循环调用方法 Add()
	// 2.函数在入栈时就已经算好了s.Add(1)
	defer s.Add(1).Add(2)
	s.Add(3)
}
```
运行结果：1 3 2
知识点：defer的入栈。defer入栈的是函数的指针，即要先初始化函数内部的数据。

## 第四题
```go
func Test08(t *testing.T) {
	defer func() { fmt.Println("打印前") }()
	defer func() { fmt.Println("打印中") }()
	defer func() { fmt.Println("打印后") }()
	panic("触发异常")
}
```
运行结果
```
打印后
打印中
打印前
panic: 触发异常
```
知识点：defer 的执行顺序是后进先出。当出现 panic 语句的时候，会先按照 defer 的后进先出的顺序执行，最后才会执行panic

## 第五题
```go
func Test071(t *testing.T) {
	x := []int{0, 1, 2}
	y := [3]*int{}
	for i, v := range x {
		defer func() { // 三个defer
			print(v)
		}()
		y[i] = &v
	}
	fmt.Println("##", *y[0], *y[1], *y[2])
	
}
```
运行结果：222 222

知识点：defer的入栈，记录的是指针

## 第六题
```go
// recover基础
func Test061(t *testing.T) {
	defer func() {
		recover() // recover() 必须在 defer() 函数中直接调用才有效。
	}()
	panic(1)
	// 1
}

// recover嵌套
func Test062(t *testing.T) {
	defer func() {
		fmt.Print(recover()) // 再次捕获panic(1)
	}()

	defer func() {
		defer fmt.Print(recover()) // 捕获上一层panic(2)
		panic(1)                   // 随后又panic
	}()
	defer recover() // 捕获无效，没有放到defer函数中
	panic(2)
	// 21
}

func Test063(t *testing.T) {
	defer func() {
		fmt.Print(recover()) // 捕获2
	}()
	defer func() {
		defer func() {
			fmt.Print(recover()) // 捕获1，在调用 defer() 时，便会计算函数的参数并压入栈中
		}()
		panic(1)
	}()
	defer recover() // 捕获无效，没有放到defer函数中
	panic(2)
	// 12
}
```
知识点：defer 和 recover