---
title: Python Numpy库
date: 2019-06-20 12:08:25
categories: 
- 编程技术
tags: 
- Python
- Numpy
- Array
---

Numpy是一个用于数组计算的Python库

ndarray对象: 表示一个N维的数组对象(多维数组、矩阵)

## 基本概念
  
  所有的数组元素具有相同的数据类型，在内存中占有相同大小的区域
  数组的维数称为秩rank
  每一个线性的数组称为是一个轴axis，表示沿着第axis个下标变化的方向进行操作；axis=0表示对每一列进行操作，axis=1表示对每一行进行操作

## 基本属性

- ndim: 数组的秩
- size: 数组元素的总个数 m*n
- dtype: 数组元素的数据类型 int32 float complex等
- shape: 数组的形状 (m, n) m行n列
- strides: 在对数组进行遍历时，跨越到下一个维度时需要跨国的字节数 (m, n) 跨越到下一行需要跨过m个字节，跨越到下一列需要跨过n个字节；通过该字段，可以快速的在多维数组中进行遍历

## 基本操作

    ```python
    import numpy as np
    # 创建一维数组
    a = np.array([1, 2, 3])
    # 创建二维数组(矩阵)
    b = np.array([[1, 2, 3],[4, 5, 6]])
    # 创建一个指定形状、数据类型、未初始化的数组
    c = np.empty([3, 2], dtype=int)
    # 创建一个指定形状，数据类型、数组元素为0的数组
    d = np.zeros([4, 5], dtype=float)
    # 创建一个指定形状，数据类型，数组元素为1的数组
    f = np.ones([2, 3], dtype=int)
    # 从已存在的列表、元祖、多维数组等创建数组
    g = np.asarray((1, 2, 3))
    # 从数值范围创建数组
    h = np.arange(10, 20, 2, dtype=float)
    i = np.arange(5)
    # 创建等差数列的一维数组
    j = np.linspace(5, 100, 20)
    # 创建等比数列的一维数组
    k = np.logspace(5, 100, 20)
    # 使用索引和切片来访问和修改数组元素
    # 使用start:stop:step格式来进行切片操作
    # 对于切片的参数格式，如果只放置一个参数，如[2]，将返回与该索引相对应的单个元素
    # 如果为[2:]，表示从该索引开始以后的所有项都将被提取
    # 如果使用了两个参数，如[2:7]，那么则提取两个索引(不包括停止索引)之间的项
    l = np.arange(10)
    m = l[2:7:2]
    # 对于多维数组，同样适用上述切片操作
    n = np.array([[1,2,3],[3,4,5],[4,5,6]])
    o = n[1:]
    # 切片操作中，还可以使用...表示选择元祖的长度与数组的维度相同
    p = n[..., 1] # 选择第二列元素
    q = n[1, ...] # 选择第二行元素
    r = n[..., 1:] # 选择第二列及其以后列的所有元素
    # 如果两个数组的shape相同，那么数组运算的结果就是两个数组对应位置上的元素运算的结果
    a = np.array([1,2,3,4])
    b = np.array([10,20,30,40])
    c = a * b
    # 当运算中的两个数组shape不同时，运算操作将触发广播机制
    a = np.array([[ 0, 0, 0],[10,10,10],[20,20,20],[30,30,30]])
    b = np.array([1,2,3])
    c = a + b
    # 对数组元素进行迭代
    a = np.arange(6).reshape(2, 3)
    for x in np.nditer(a):
      print(x)
    # 可以控制遍历的顺序
    for x in np.nditer(a, order="C"): # 行序优先
      print(x)
    for x in np.nditer(a, order="F"): # 列序优先
      print(x)
    # 在不改变数据的情况下调整数组的形状
    e = np.reshape(d, [5, 4])
    # 数组的flat属性为数组元素迭代器
    a = np.arange(9).reshape(3,3)
    for x in a.flat:
      print (x)
    # 数组的flatten方法返回展平的数组元素，是对原始数组元素的拷贝，对其所做的修改不会影响原始数组
    # 数组的ravel方法返回展示的数组元素，是对原始数组元素的引用，对其所做的修改会影响原始数组
    a = np.arange(8).reshape(2,4)
    b = a.flatten()
    c = a.ravel()
    # 转换数组的维度 转置
    a = np.arange(12).reshape(3,4)
    b = np.transpose(a)
    # 数组的T属性即为数组的转置
    a = np.arange(12).reshape(3,4)
    b = a.T
    # 沿指定轴连接相同shape的多个数组
    a = np.array([[1,2],[3,4]])
    b = np.array([[5,6],[7,8]])
    c = np.concatenate((a, b), axis=0)
    d = np.concatenate((a, b), axis=1)
    # 沿新轴连接相同shape的多个数组，会增加新的维度
    a = np.array([[1,2],[3,4]])
    b = np.array([[5,6],[7,8]])
    c = np.stack((a, b), axis=0) # 新增维度的下标为0
    d = np.stack((a, b), axis=1) # 新增维度的下标为1
    # 水平或垂直堆叠来生成新的数组
    a = np.array([[1,2],[3,4]])
    b = np.array([[5,6],[7,8]])
    c = np.hstack((a, b))
    d = np.vstack((a, b))
    # 将数组沿着指定的轴分割为子数组
    a = np.arange(9)
    b = np.split(a, 3)
    c = np.split(a, [4, 7])
    # 水平或垂直分割数组为子数组
    a = np.floor(10 * np.random.random((2, 6)))
    b = np.hsplit(a, 3) # 水平分割为三个子数组
    c = np.vsplit(a, 2) # 垂直分割为两个子数组
    # 返回指定shape的新数组，如果新数组的大小大于原数组，则填入原数组中数据元素的副本
    a = np.array([[1,2,3],[4,5,6]])
    b = np.resize(a, (3, 2))
    c = np.resize(a, (3, 3))
    # 在数组的末尾添加值
    a = np.array([[1,2,3],[4,5,6]])
    b = np.append(a, [[7, 8, 9]], axis=0)
    c = np.append(a, [[7, 8], [9, 10]], axis=1)
    # 去除数组元素中的重复值
    a = np.array([5,2,6,2,7,5,6,8,2,9])
    b = np.unique(a)
    # 三角函数运算 sin cos tan
    a = np.array([0,30,45,60,90])
    b = np.sin(a*np.pi/180)
    # 舍入函数
    a = np.array([1.0,5.55,  123,  0.567,  25.532])
    b = np.around(a)
    c = np.floor(a)
    d = np.ceil(a)
    # 算术函数 add subtract multiply divide 对数组的shape有要求
    a = np.arange(9, dtype = np.float_).reshape(3,3)
    b = np.array([10,10,10])
    c = np.add(a, b)
    d = np.subtract(a, b)
    e = np.multiply(a, b)
    f = np.divide(a, b)
    # 统计函数
    # amin amax 获取沿着指定轴的最小和最大值
    a = np.array([[3,7,5],[8,4,3],[2,4,9]])
    b = np.amin(a, axis=0)
    c = np.amin(a, axis=1)
    # ptp 获取沿着指定轴的最大最小值的差
     a = np.array([[3,7,5],[8,4,3],[2,4,9]])
    b = np.ptp(a, axis=0)
    c = np.ptp(a, axis=1)
    # percentile 获取沿着指定轴的百分位数
    a = np.array([[10, 7, 4], [3, 2, 1]])
    b = np.percentile(a, 50) # 所有元素排序排序之后的中位数，如果axis参数为None，则默认展开数组
    c = np.percentile(a, 50, axis=0) # 在纵轴上求
    c = np.percentile(a, 50, axis=1) # 在横轴上求
    # median 获取沿着指定轴的中位数
    a = np.array([[30,65,70],[80,95,10],[50,90,60]])
    b = np.median(a) # 所有元素排序排序之后的中位数
    c = np.percentile(a, axis=0) # 在纵轴上求
    c = np.percentile(a, axis=1) # 在横轴上求
    # mean 获取沿着指定轴的平均值
    # average 获取沿着指定轴的加权平均值
    # std 获取标准差
    # var 获取方差
    # 排序操作 sort quicksort快排 mergesort归并 heapsort堆排序
    a = np.array([[3,7],[9,1]])
    b = np.sort(a, axis=0, kind="quicksort") # 按照纵轴进行排序
    # argsort 返回数组值从小到大的索引值
    a = np.array([66, 55, 77])  
    b = np.argsort(a)
    # 数组元素条件筛选函数
    # argmax argmin 获取沿着指定轴中最大和最小元素的索引
    # nonzero 获取数组中非零元素的索引
    # where 获取数组中满足给定条件的元素的索引
    # extract 根据某个条件从数组中抽取元素，返回这些元素的索引
    # 副本与视图，副本是数据的完整拷贝，对其进行的修改不影响原始数据；视图是对数据的引用，对其进行的修改会影响原始数据
    # 副本发生在: 1.使用np.copy函数
    # 视图发生在: 1.数组的切片操作 2.使用np.view()函数
    ```
