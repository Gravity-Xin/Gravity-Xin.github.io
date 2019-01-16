---
title: Golang-container
date: 2019-01-15 17:35:52
categories:
- Golang
tags:
- 
- goroutine
- channel
- concurrency
- select
- range
comments: true
---

## container/heap

实现了堆数据结构: [heap](https://gravity-xin.github.io/2018/11/12/Heap/)

提供了`heap.Interface`接口，同时提供了`Init、Push、Pop、Remove、Fix`方法对堆对象进行操作

```Go
// Interface 堆对象需要实现的接口
type Interface interface {
    sort.Interface // 排序接口
    Push(x interface{}) // 向堆中加入元素，作为第Len()个元素
    Pop() interface{}   // 从堆中删除元素，并返回Len()-1个元素
}

// Init 对提供的满足heap.Interface接口的对象进行建立最小堆的操作
// 时间复杂度为O(h.Len())
func Init(h Interface) {
    n := h.Len()
    for i := n/2 - 1; i >= 0; i-- {
        down(h, i, n) // 对堆中的元素下溯，调整堆
    }
}

// Push 向堆底中增加元素，并通过上溯调整堆
// 时间复杂度O(log(h.Len()))
func Push(h Interface, x interface{}) {
    h.Push(x)
    up(h, h.Len()-1) // 对堆中元素进行上溯，调整堆
}

// Pop 从堆顶删除元素，将堆尾元素放入堆顶，并通过下溯调整堆
// 时间复杂度为时间复杂度O(log(h.Len()))
func Pop(h Interface) interface{} {
    n := h.Len() - 1
    h.Swap(0, n) // 将堆尾部元素放入堆顶
    down(h, 0, n) // 对堆中元素下溯吗，调整堆
    return h.Pop()
}

// Remove 从堆中任意位置(数组下标)删除元素
// 时间复杂度为时间复杂度O(log(h.Len()))
func Remove(h Interface, i int) interface{} {
    n := h.Len() - 1
    if n != i {
        h.Swap(i, n)
        if !down(h, i, n) {
            up(h, i)
        }
    }
    return h.Pop()
}

// Fix 当位于i的位置的值发生变化后，调整堆
// 时间复杂度为时间复杂度O(log(h.Len()))
func Fix(h Interface, i int) {
    if !down(h, i, h.Len()) {
        up(h, i)
    }
}
```

因此，我们只需要实现heap.Interface接口，便可以通过heap包提供的方法对堆进行各种操作

基于heap实现优先级队列

```Go

// Item 堆中的元素
type Item struct {
    value    string // 元素值
    priority int    // 元素的优先级
    index    int    // 元素在堆中的索引(数组下标)
}

// PriorityQueue 实现了heap.Interface的堆对象，本质上是一个数组
type PriorityQueue []*Item

// Len 返回PriorityQueue内置数组的长度
func (pq PriorityQueue) Len() int { return len(pq) }

// Less 返回PriorityQueue中元素优先级的比较结果
func (pq PriorityQueue) Less(i, j int) bool {
    return pq[i].priority > pq[j].priority // 最大堆
}

// Swap 对PriorityQueue中的任意两个元素进行替换
func (pq PriorityQueue) Swap(i, j int) {
    pq[i], pq[j] = pq[j], pq[i]
    pq[i].index = i // 更新元素索引
    pq[j].index = j
}

// Push 向PriorityQueue添加元素，放入数组末尾
func (pq *PriorityQueue) Push(x interface{}) {
    n := len(*pq)
    item := x.(*Item)
    item.index = n
    *pq = append(*pq, item)
}

// Pop 从PriorityQueue删除元素，删除数组尾部元素
func (pq *PriorityQueue) Pop() interface{} {
    old := *pq
    n := len(old)
    item := old[n-1]
    item.index = -1
    *pq = old[0 : n-1]
    return item
}

// 通过heap.Fix方法处理堆中元素优先级变化的情况
func (pq *PriorityQueue) update(item *Item, value string, priority int) {
    item.value = value
    item.priority = priority
    heap.Fix(pq, item.index)
}
```

## container/list

实现了双向链表的数据结构

提供了`list.New()`方法来创建双向链表，同时链表还提供了`Init、Back、Front、InsertAfter、InsertFront、Len`等链表操作方法

```Go
// Element 双向链表中的元素
type Element struct {
    // 当前元素的前一个和后一个元素 next.Front()=prev.Back()=e
    // 实际是一个环
    next, prev *Element

    // 当前元素所属的链表对象
    list *List

    // 当前元素存储的实际值
    Value interface{}
}

// Next 返回元素在链表中的下一个元素或者返回nil
func (e *Element) Next() *Element {
    if p := e.next; e.list != nil && p != &e.list.root {
        return p
    }
    return nil
}

// Prev 返回元素在链表中的前一个元素或者返回nil
func (e *Element) Prev() *Element {
    if p := e.prev; e.list != nil && p != &e.list.root {
        return p
    }
    return nil
}

// List 双向链表
type List struct {
    root Element // root元素，作为哨兵 root.prev链表尾部元素 root.next链表头部元素
    len  int     // 链表长度，不包含root元素
}

// Init 初始化或者清空链表
func (l *List) Init() *List {
    l.root.next = &l.root
    l.root.prev = &l.root
    l.len = 0
    return l
}

// New 返回一个已初始化的链表对象
func New() *List { return new(List).Init() }

// Len 返回链表的长度
// 时间复杂度是O(1)
func (l *List) Len() int { return l.len }

// Front 返回链表的头部元素，如果链表为空则返回nil
func (l *List) Front() *Element {
    if l.len == 0 {
        return nil
    }
    return l.root.next
}

// Back 返回链表的尾部元素，如果链表为空则返回nil
func (l *List) Back() *Element {
    if l.len == 0 {
        return nil
    }
    return l.root.prev
}

// Remove 从链表中删除元素e，返回元素e中的值
func (l *List) Remove(e *Element) interface{} {
    if e.list == l {
        l.remove(e)
    }
    return e.Value
}

// PushFront 在链表头部插入值
func (l *List) PushFront(v interface{}) *Element {
    l.lazyInit()
    return l.insertValue(v, &l.root)
}

// PushBack 在链表尾部插入值
func (l *List) PushBack(v interface{}) *Element {
    l.lazyInit()
    return l.insertValue(v, l.root.prev)
}

// InsertBefore 在元素mark之前插入值
func (l *List) InsertBefore(v interface{}, mark *Element) *Element {
    if mark.list != l {
        return nil
    }
    return l.insertValue(v, mark.prev)
}

// InsertAfter 在元素mark之后插入值
func (l *List) InsertAfter(v interface{}, mark *Element) *Element {
    if mark.list != l {
        return nil
    }
    return l.insertValue(v, mark)
}

// MoveToFront 将链表中的某个元素插入到链表头部
func (l *List) MoveToFront(e *Element) {
    if e.list != l || l.root.next == e {
        return
    }
    l.insert(l.remove(e), &l.root)
}

// MoveToBack 将链表中的某个元素插入到链表尾部
func (l *List) MoveToBack(e *Element) {
    if e.list != l || l.root.prev == e {
        return
    }
    // see comment in List.Remove about initialization of l
    l.insert(l.remove(e), l.root.prev)
}

// MoveBefore 将链表中的元素插入到元素mark之前
func (l *List) MoveBefore(e, mark *Element) {
    if e.list != l || e == mark || mark.list != l {
        return
    }
    l.insert(l.remove(e), mark.prev)
}

// MoveAfter 将链表中的元素插入到元素mark之后
func (l *List) MoveAfter(e, mark *Element) {
    if e.list != l || e == mark || mark.list != l {
        return
    }
    l.insert(l.remove(e), mark)
}

// PushBackList 将另一个链表中的元素依次插入到当前链表的尾部
func (l *List) PushBackList(other *List) {
    l.lazyInit()
    for i, e := other.Len(), other.Front(); i > 0; i, e = i-1, e.Next() { // 对链表进行遍历
        l.insertValue(e.Value, l.root.prev)
    }
}

// PushFrontList 将另一个链表中的元素依次插入到当前链表的头部
func (l *List) PushFrontList(other *List) {
    l.lazyInit()
    for i, e := other.Len(), other.Back(); i > 0; i, e = i-1, e.Prev() {
        l.insertValue(e.Value, &l.root)
    }
}
```

## container/ring

实现了环形链表的数据结构

提供了`ring.New()`来创建环形链表，同时链表还提供了`Next、Prev、Move`等方法对链表进行操作

```Go
// Ring 环形链表元素，同时也是环形链表对象
// 环形链表没有头部或尾部元素，可以从环中任一个元素开始对整个环进行遍历
type Ring struct {
    next, prev *Ring
    Value      interface{} // 元素的具体值
}

// Next 获取环中当前元素的下一个元素
func (r *Ring) Next() *Ring {
    if r.next == nil {
        return r.init()
    }
    return r.next
}

// Prev 获取环中当前元素的前一个元素
func (r *Ring) Prev() *Ring {
    if r.next == nil {
        return r.init()
    }
    return r.prev
}

// Move 返回环形链表中后面 (n > 0) 或者前面 (n < 0) 第 n % r.Len() 个元素
func (r *Ring) Move(n int) *Ring {
    if r.next == nil {
        return r.init()
    }
    switch {
    case n < 0:
        for ; n < 0; n++ {
            r = r.prev
        }
    case n > 0:
        for ; n > 0; n-- {
            r = r.next
        }
    }
    return r
}

// New 创建包含n个元素的环形链表
func New(n int) *Ring {
    if n <= 0 {
        return nil
    }
    r := new(Ring)
    p := r
    for i := 1; i < n; i++ {
        p.next = &Ring{prev: p}
        p = p.next
    }
    p.next = r // 首尾相连
    r.prev = p
    return r
}

// Link 将环形链表r和s相连接
// 如果r和s为同一个环形链表，则删除r和s中之间的元素，并返回被删除的元素构成的子环形链表
// 如果r和s不为同一个环形链表，则创建一个新的环形链表使得 r.Next() = s
func (r *Ring) Link(s *Ring) *Ring {
    n := r.Next()
    if s != nil {
        p := s.Prev()
        // Note: Cannot use multiple assignment because
        // evaluation order of LHS is not specified.
        r.next = s
        s.prev = r
        n.prev = p
        p.next = n
    }
    return n
}

// Unlink 删除环形链表中 n % r.Len() 个元素,从r.Next()开始，返回的是被删除的环形链表
func (r *Ring) Unlink(n int) *Ring {
    if n <= 0 {
        return nil
    }
    return r.Link(r.Move(n + 1))
}

// Len 返回环形链表的长度
// 时间复杂度o(n)
func (r *Ring) Len() int {
    n := 0
    if r != nil {
        n = 1
        for p := r.Next(); p != r; p = p.next {
            n++
        }
    }
    return n
}

// Do 向前遍历环形链表的元素，并元素值执行f函数操作
func (r *Ring) Do(f func(interface{})) {
    if r != nil {
        f(r.Value)
        for p := r.Next(); p != r; p = p.next {
            f(p.Value)
        }
    }
}
```