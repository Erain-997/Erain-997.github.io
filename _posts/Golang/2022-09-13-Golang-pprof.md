---
layout:     post
title:      Golang-pprof
subtitle:   pprof性能分析
date:       2022-09-13
author:     Erain
header-img: img/home-bg.jpg
categories: Golang
catalog: true
tags:
    - Golang
---

# 真正分析时常用4种

- **CPU Profiling**：CPU 分析，按照一定的频率采集所监听的应用程序 CPU（含寄存器）的使用情况，可确定应用程序在主动消耗 CPU 周期时花费时间的位置
- **Memory Profiling**：内存分析，在应用程序进行堆分配时记录堆栈跟踪，用于监视当前和历史内存使用情况，以及检查内存泄漏
- **Block Profiling**：阻塞分析，记录 goroutine 阻塞等待同步（包括定时器通道）的位置
- **Mutex Profiling**：互斥锁分析，报告互斥锁的竞争情况