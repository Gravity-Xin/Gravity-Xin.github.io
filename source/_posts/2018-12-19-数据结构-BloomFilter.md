---
title: 数据结构-BloomFilter
date: 2018-12-19 17:25:17
categories: 
- 编程技术
tags: 
- BloomFilter
comments: true
---

## 定义

对于包含大量元素的集合，需要一种能够判断某个元素是否属于该集合的方法

* 使用位数组，降低内存占用
* 使用多个相互独立的Hash函数
* 如果元素在集合中，那么一定能够被检查出来；如果元素不在集合中，有可能会被误报为在集合中

* 错误率
* 位数组长度
* 哈希函数的数量

## 实现

```Go
//BloomFilter 布隆过滤器接口
type BloomFilter interface {
    Add([]byte) // 添加元素
    Exist([]byte) bool // 判断元素是否存在
}

type bloomFilter struct {
    bitset  *bitset.BitSet
    length  uint
    hashNum uint
}

func (bl *bloomFilter) Add(data []byte) {
    h := baseHash(data)
    for i := uint(0); i < bl.hashNum; i++ {
        index := location(h, uint64(i)) % uint64(bl.length)
        bl.bitset.Set(uint(index))
    }
}

func (bl *bloomFilter) Exist(data []byte) bool {
    h := baseHash(data)
    for i := uint(0); i < bl.hashNum; i++ {
        index := location(h, uint64(i)) % uint64(bl.length)
        if !bl.bitset.Test(uint(index)) { // 如果该位未被填充，则元素一定不存在
            return false
        }
    }
    return true // 返回元素存在，有可能元素并不存在，计算出来的index被其他元素填充了
}

//NewBloomFilter 创建布隆过滤器
// k 位数组长度 m hash函数个数
func NewBloomFilter(k, m uint) BloomFilter {
    bl := bloomFilter{
        bitset:  bitset.New(k),
        length:  k,
        hashNum: m,
    }
    return &bl
}

func baseHash(data []byte) [4]uint64 { // 获取四个hash值
    h1 := sha256.New()
    h1.Write(data)
    length := h1.Size()
    sum := h1.Sum(nil)
    r := [4]uint64{}
    for i := 0; i < 4; i++ {
        if 4*i+7 < length {
            buf := []byte{sum[4*i], sum[4*i+1], sum[4*i+2], sum[4*i+3], sum[4*i+4], sum[4*i+5], sum[4*i+6], sum[4*i+7]}
            r[i] = bytesToInt64(buf)
        }
    }
    return r
}

func bytesToInt64(buf []byte) uint64 { // byte数组转uint64
    return uint64(binary.BigEndian.Uint64(buf))
}

func location(h [4]uint64, i uint64) uint64 { // 计算第i个hash的hash值
    return h[i%2] + i*h[2+(((i+(i%2))%4)/2)]
}

func main() {
    bl := NewBloomFilter(65536, 5)
    bl.Add([]byte("haha"))
    fmt.Println(bl.Exist([]byte("haha")))
    fmt.Println(bl.Exist([]byte("xixi")))
}
```