---
layout:     post
title:      Golang-sort包
subtitle:   sort包基础用法，基础原理
date:       2022-06-10
author:     Erain
header-img: img/home-bg.jpg
categories: Golang
catalog: true
tags:
    - Golang
    - Go包
---

sort学习记录

# sort的基础使用

对于一些稍复杂一些的排序，可以使用Less自定义排序

```go
func TestSort(t *testing.T) {
	a := []int{1, 2, 3, 4, 6}
	fmt.Println(sort.SearchInts(a, 1))                                      // 默认靠前一位
	fmt.Println(sort.SearchInts(a, 5))                                      // 默认升序
	fmt.Println(sort.Search(len(a), func(i int) bool { return a[i] >= 0 })) // 升序
	b := []int{9, 8, 7, 6, 5, 4}
	fmt.Println(sort.Search(len(b), func(i int) bool { return b[i] <= 5 })) // 降序

	data1 := []int{74, 59, 238, -784, 9845, 959, 905, 0, 0, 42, 7586, -5467984, 7586}
	c := sort.IntSlice(data1) //conversion to type IntArray
	sort.Sort(c)              // 基于快速排序
	fmt.Println("The array is sort", sort.IsSorted(sort.Reverse(sort.IntSlice(b))))
	fmt.Println("The array is sort", sort.IntsAreSorted(c)) // 只校验升序，只要是指定类型的都是默认升序
	fmt.Printf("The sorted array is: %v\n", c)

	// 自定义Less排序比较器
	people := []struct {
		Name string
		Age  int
	}{
		{"Gopher", 7},
		{"Alice", 55},
		{"Vera", 24},
		{"Bob", 75},
	}
	sort.Slice(people, func(i, j int) bool { return people[i].Name < people[j].Name })
	fmt.Println("By name:", people)
	sort.Slice(people, func(i, j int) bool { return people[i].Age < people[j].Age })
	fmt.Println("By age:", people)
}
```

# sort实现的功能
- 对基本数据类型切片的排序支持
- 自定义 Less 排序比较器
- 自定义数据结构的排序
- 判断基本数据类型切片是否已经排好序
- 基本数据元素查找

sort的排序方式源码，值得一看: 

注意这是不稳定排序。稳定排序的定义是:
> 待排序的记录序列中可能存在两个或两个以上关键字相等的记录。排序前的序列中Ri领先于Rj（即i<j）.若在排序后的序列中Ri仍然领先于Rj，则称所用的方法是稳定的。比如int数组[1,1,1,6,4]中a[0],a[1],a[2]的值相等，在排序时不改变其序列，则称所用的方法是稳定的。
```go
// Sort sorts data.
// It makes one call to data.Len to determine n, and O(n*log(n)) calls to
// data.Less and data.Swap. The sort is not guaranteed to be stable.
func Sort(data Interface) {
	n := data.Len()
	quickSort(data, 0, n, maxDepth(n))
}
```

sort 包中也提供了稳定的排序，通过调用**sort.Stable**来实现

# golang内置的排序



- **插入排序**	*func insertionSort(data Interface, a, b int)*
- **堆排序**	*func heapSort(data Interface, a, b int)*
- **快速排序**	*func quickSort(data Interface, a, b, maxDepth int)*
- **归并排序**	*func symMerge(data Interface, a, m, b int)*

```go
func quickSort(data Interface, a, b, maxDepth int) {
	for b-a > 12 { // Use ShellSort for slices <= 12 elements
	    // maxDepth 返回快速排序应该切换的阈值
        // 当 maxDepth为0的时候进行堆排序
		if maxDepth == 0 {
			heapSort(data, a, b)// 进行堆排序
			return
		}
		maxDepth--
		// doPivot 是快排核心算法，它取一点为轴，把不大于轴的元素放左边，大于轴的元素放右边，返回小于轴部分数据的最后一个下标，以及大于轴部分数据的第一个下标
        // 下标位置 a...mlo,pivot,mhi...b
        // data[a...mlo] <= data[pivot]
        // data[mhi...b] > data[pivot]
        // 和中位数一样的数据就不用在进行交换了，维护这个范围值能减少数据的次数  
		mlo, mhi := doPivot(data, a, b)
		// 避免递归过深
        // 循环是比递归节省时间的，如果有大规模的子节点，让小的先递归，达到了 maxDepth 也就是可以触发堆排序的条件了，然后使用堆排序进行排序
		if mlo-a < b-mhi {
			quickSort(data, a, mlo, maxDepth)
			a = mhi // i.e., quickSort(data, mhi, b)
		} else {
			quickSort(data, mhi, b, maxDepth)
			b = mlo // i.e., quickSort(data, a, mlo)
		}
	}
	// 如果切片的长度大于1小于等于12的时候，使用 shell 排序
	if b-a > 1 {
		// shell 排序
		// Do ShellSort pass with gap 6
		// It could be written in this simplified form cause b-a <= 12
		for i := a + 6; i < b; i++ {
			if data.Less(i, i-6) {
				data.Swap(i, i-6)
			}
		}
		// 进行插入排序
		insertionSort(data, a, b)
	}
}

// maxDepth returns a threshold at which quicksort should switch
// to heapsort. It returns 2*ceil(lg(n+1)).
func maxDepth(n int) int {
	var depth int
	for i := n; i > 0; i >>= 1 {
		depth++
	}
	return depth * 2
}

// doPivot 是快排核心算法，它取一点为轴，把不大于轴的元素放左边，大于轴的元素放右边，返回小于轴部分数据的最后一个下标，以及大于轴部分数据的第一个下标
// 下标位置 lo...midlo,pivot,midhi...hi
// data[lo...midlo] <= data[pivot]
// data[midhi...hi] > data[pivot]
func doPivot(data Interface, lo, hi int) (midlo, midhi int) {
	m := int(uint(lo+hi) >> 1) // Written like this to avoid integer overflow.
	if hi-lo > 40 {
		// Tukey's ``Ninther,'' median of three medians of three.
		// 文章链接 https://www.johndcook.com/blog/2009/06/23/tukey-median-ninther/
		// 通过该算法求出中位数
		s := (hi - lo) / 8
		medianOfThree(data, lo, lo+s, lo+2*s)
		medianOfThree(data, m, m-s, m+s)
		medianOfThree(data, hi-1, hi-1-s, hi-1-2*s)
	}
	// 求出中位数 data[m] <= data[lo] <= data[hi-1]
	medianOfThree(data, lo, m, hi-1)

	// Invariants are:
	//	data[lo] = pivot (set up by ChoosePivot)
	//	data[lo < i < a] < pivot
	//	data[a <= i < b] <= pivot
	//	data[b <= i < c] unexamined
	//	data[c <= i < hi-1] > pivot
	//	data[hi-1] >= pivot
	// 中位数
	pivot := lo
	a, c := lo+1, hi-1

	// 处理使 data[a] < pivot
	for ; a < c && data.Less(a, pivot); a++ {
	}
	b := a
	for {
		for ; b < c && !data.Less(pivot, b); b++ { // data[b] <= pivot
		}
		for ; b < c && data.Less(pivot, c-1); c-- { // data[c-1] > pivot
		}
		// 左边和右边重合或者已经在右边的右侧
		if b >= c {
			break
		}
		// data[b] > pivot; data[c-1] <= pivot
		// 左侧的数据大于右侧，交换，然后接着排序
		data.Swap(b, c-1)
		b++
		c--
	}
	// If hi-c<3 then there are duplicates (by property of median of nine).
	// Let's be a bit more conservative, and set border to 5.
	// 如果 hi-c<3 则存在重复项（按中位数为 9 的属性）。
    // 让我们稍微保守一点，将边框设置为 5。
 
    // 因为c为划分pivot的大小的临界值，所以在9值划分时，正常来说，应该是两边各4个
    // 由于左边是<=，多了个相等的情况，所以5，3分布，也是没有问题
    // 如果hi-c<3，c的值明显偏向于hi，说明有多个和pivot重复值
    // 为了更保守一点，所以设置为5(反正只是多校验一次而已)
	protect := hi-c < 5
	// 对比hi-c是否小于总数量的1/4
	if !protect && hi-c < (hi-lo)/4 {
		// Lets test some points for equality to pivot
		dups := 0
		if !data.Less(pivot, hi-1) { // data[hi-1] = pivot
			data.Swap(c, hi-1)
			c++
			dups++
		}
		if !data.Less(b-1, pivot) { // data[b-1] = pivot
			b--
			dups++
		}
		// m-lo = (hi-lo)/2 > 6
		// b-lo > (hi-lo)*3/4-1 > 8
		// ==> m < b ==> data[m] <= pivot
		if !data.Less(m, pivot) { // data[m] = pivot
			data.Swap(m, b-1)
			b--
			dups++
		}
		// if at least 2 points are equal to pivot, assume skewed distribution
		// 如果上面的 if 进入了两次， 就证明现在是偏态分布（也就是左右不平衡的）
		protect = dups > 1
	}
	// 不平衡，接着进行处理
    // 这里划分的是<pivot和=pivot的两组
	if protect {
		// Protect against a lot of duplicates
		// Add invariant:
		//	data[a <= i < b] unexamined
		//	data[b <= i < c] = pivot
		for {
			for ; a < b && !data.Less(b-1, pivot); b-- { // data[b] == pivot
			}
			for ; a < b && data.Less(a, pivot); a++ { // data[a] < pivot
			}
			if a >= b {
				break
			}
			// data[a] == pivot; data[b-1] < pivot
			data.Swap(a, b-1)
			a++
			b--
		}
	}
	// Swap pivot into middle
	// 交换中位数到中间
	data.Swap(pivot, b-1)
	return b - 1, c
}
```

# 自定义数据类型排序

复写sort比较器

```go
type Person struct {
	Name string
	Age  int
}
type ByAge []Person

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }

func TestSortStruct(t *testing.T) {
	people := []Person{
		{"Gopher", 7},
		{"Alice", 55},
		{"Vera", 24},
		{"Bob", 75},
	}

	sort.Sort(ByAge(people))
	fmt.Println(people)
}
```

# 总结
对于这几种排序算法的使用，sort 包中是混合使用的，sort 对于排序算法的实现，是结合了多种算法，最终实现了一个高性能的排序算法。

1. 如果切片长度大于12的时候使用快排，使用快排的时候，如果满足了使用堆排序的条件没这个排序对于后面的数据的处理，又会转换成堆排序；

2. 切片长度小于12了，就使用 shell 排序，shell 排序只处理一轮数据，后面数据的排序使用插入排序；

3. 堆排序和插入排序就是正常的排序处理了