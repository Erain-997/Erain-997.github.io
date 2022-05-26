---
layout:     post
title:      Code-Switch相关用法
subtitle:   当判断条件>2时，尽量使用switch美化代码。少用if-else
date:       2022-05-26
author:     Erain
header-img: img/home-bg.jpg
categories: Code-Go
catalog: true
tags:
    - Golang
    - Code
---

- 什么都不判断
```go
func Test01(t *testing.T) {
	a := rand.intn(5)
	switch  {
	case a=1:
		fmt.Println(1)
	case a=3:
		fmt.Println(3)
	case a=2:
		fmt.Println(2)
	case a=4:
		fmt.Println(4)
	}
}
```

- 通过值判断
```go
func Test01(t *testing.T) {
	a := rand.intn(5)
	switch a {
	case 1:
		fmt.Println(1)
	case 3:
		fmt.Println(3)
	case 2:
		fmt.Println(2)
	case 4:
		fmt.Println(4)
	}
}
```

- 穿透
```go
func Test01(t *testing.T) {
	a := rand.intn(5)
	switch a {
	case 1:
		fmt.Println(1)
		fallthrough // 穿透，即如果执行case1，会接着执行case2
	case 3:
		fmt.Println(3)
	case 2:
		fmt.Println(2)
	case 4:
		fmt.Println(4)
	}
}
```

- 通过类型判断
```go
func Test01(t *testing.T) {
	var b interface{}
	b = int(2)
	switch b.(type) {
	case int:
		fmt.Println(3)
	case string:
		fmt.Println(2)
	}
}
```