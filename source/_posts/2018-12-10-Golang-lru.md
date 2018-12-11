---
title: Golang-lru
date: 2018-12-10 17:59:04
categories:
- Golang
tags:
- lru
comments: true
---

## LRU简介

LRU(Least Recent Used): 最近最少使用，一种缓存置换算法
基于的原理: 最近被使用的缓存，后续被使用的可能性更大
实现方式:
    对于使用的缓存按照访问时间进行排序，对数据的读写都会更新其访问时间
    当缓存已满时，驱逐出访问时间最早的数据，然后将数据写入
    以上可以保证缓存中的数据为热点数据，提高缓存命中率

## 实现

```Go
// MemCache 缓存对象
type MemCache struct {
    mtx        sync.RWMutex
    capacity   int                           //固定缓存大小
    data       map[interface{}]*list.Element //缓存数据
    lru        *list.List                    //按照最近访问时间排序的LRU列表
    hits, gets int32                         //记录缓存的访问和命中次数
}

// MemCacheStatus 缓存状态
type MemCacheStatus struct {
    capacity int
    size     int
    gets     int32
    hits     int32
}

// 存储的KV对象
type entry struct {
    key   interface{}
    value interface{}
}

//NewMemCache 创建缓存对象
func NewMemCache(capacity int) *MemCache {
    return &MemCache{
        capacity: capacity,
        data:     make(map[interface{}]*list.Element),
        lru:      list.New(),
    }
}

//Set 更新缓存
//首先查询Key是否存在，如果存在则直接替换并在lru中将Key移动到队头
//如果Key不存在，则判断缓存是否已满，如果未满，则直接写入，并在lru的队头创建key
//如果缓存已满，则将lru队尾位置的key移除，然后再写入，并在lru队头创建key
func (m *MemCache) Set(key, value interface{}) {
    m.mtx.Lock()
    defer m.mtx.Unlock()

    if ele, ok := m.data[key]; ok {
        m.lru.MoveToFront(ele)
        ele.Value.(*entry).value = value
        return
    }

    if len(m.data) == m.capacity {
        backEle := m.lru.Back()
        m.lru.Remove(backEle)
        delete(m.data, backEle.Value.(*entry).key)
    }

    newEle := m.lru.PushFront(&entry{key: key, value: value})
    m.data[key] = newEle
}

//Get 获取缓存
//如果缓存存在，则将key移动到队头
func (m *MemCache) Get(key interface{}) (value interface{}) {
    m.mtx.RLock()
    defer m.mtx.RUnlock()
    atomic.AddInt32(&m.gets, 1)
    if ele, ok := m.data[key]; ok {
        m.lru.MoveToFront(ele)
        value = ele.Value.(*entry).value
        atomic.AddInt32(&m.hits, 1)
    }
    return
}

//Status 获取缓存状态
func (m *MemCache) Status() MemCacheStatus {
    return MemCacheStatus{
        capacity: m.capacity,
        size:     len(m.data),
        gets:     m.gets,
        hits:     m.hits,
    }
}
```