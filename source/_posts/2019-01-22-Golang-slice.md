---
title: Golang-slice
date: 2019-01-22 17:25:33
categories:
- Golang
tags:
- slice
- len
- cap
comments: true
---

golang中的slice(切片)是基于array(数组)上的内置类型，可以看做是能够自动扩容的动态数组
与内置的map, chan一样，都是引用类型

## 使用方法

- array
  - 需要指定数组元素类型和数组长度
  - 底层为连续的内存空间，使用下标作为索引来进行存取
  - 声明后，数组元素为零值
  - 数组长度也是数组类型的一部分，因此`[3]int`和`[4]int`不是同一个类型

```Go
func main() {
    nums := [3]int{}
    nums[0] = 1
    n := nums[0]
    n = 2
    fmt.Printf("nums: %v\n", nums)
    fmt.Printf("n: %d\n", n)
}
```

- slice
  - 底层为数组，
  - 不需要指定切片的长度
  - 使用`var []T`, `[]T{}`或 `make([]T, len, cap)`来创建
  - 使用`append`来增加元素，可以自动扩容
  - 使用`copy`来进行切片的拷贝，返回被复制的元素数
  - 使用`[start:end]`来进行切片的resize操作

```Go
func main() {
    nums := [3]int{}
    nums[0] = 1
    dnums := nums[:]
    fmt.Printf("dnums: %v", dnums)
}
```

## 实现方式

![slice](/images/Golang-slice的实现.png)

```Go
type slice struct {
    array unsafe.Pointer //所引用的底层数据的指针
    len   int  // 当前切片的元素个数
    cap   int  // 当前切片的容量(底层数组的长度)
}
```

扩容带来的问题

- 未扩容时，对slice的修改会直接影响到底层的array
- 发送扩容时，slice不再引用底层数据，对slice的修改不再影响到底层的array

empty slice和nil slice

```Go
emptyNums := []int{} // 切片中的array指向一个空的数组
var nilNums []int // 切片中的array为nil，不指向任何数组
```
