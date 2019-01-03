---
title: Golang-Effective Go
date: 2019-01-03 10:20:55
categories:
- Golang
tags:
comments: true
---

## 代码格式

使用`gofmt`工具

## 注释

使用 `//`进行代码行注释
使用`/* */`进行代码块注释

使用`godoc`工具生成说明文档

## 标识符

包名: 小写，单个单词，不要与包内部导出的对象名称重复
接口名称: 以`er`结尾
变量名称: 驼峰式

## 分号

不需要在源码中添加分号，词法分析器会自动添加

## 控制语句

`if else` and `switch`
`switch t := t.(type)` 类型推断
`for`, `break`, `continue` and `range`
`select`

## 函数

多个返回值
返回值可以进行命名
`defer`进行资源回收，多个`defer`以`LIFO`的顺序执行

## 数据分配

使用`make`和`new`
`make(T, args)`用于对slice、map和chan类型进行内存分配和初始化，并可以增加参数来指定对象的初始状态，返回类型T
`new(T)`用于对其他类型进行内存分配，返回类型T的零值，返回类型为*T

使用自定义函数`NewXXX`来进行对象内存分配和初始化

## 初始化

常量: 编译期创建，可以使用`iota`进行递增
变量: 包级别变量
init函数: 包级别初始化函数

## 方法

方法接受者可以为对象或对象的指针

## 接口

鸭子类型
使用类型推断确定接口的具体类型
函数对象也可以增加方法，如`http.HandlerFunc`，从而让函数对象实现了某个接口

## 空占位符

函数包含多返回值，使用`_`忽略不需要的返回值
使用`_`来忽略未被使用的导入包

## 组合

在struct和interface中使用组合来进行扩展

## 并发

不要通过共享来通信，而是通过通信来共享
使用goroutine和channel(buffered and unbuffered)
使用带缓冲的channel来模拟信号量

## 错误处理

根据具体场景对error对象进行扩展
使用`panic`和`recover`来进行运行时异常处理和恢复

## 参考

[Effective Go](https://golang.org/doc/effective_go.html)
