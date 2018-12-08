---
title: Golang-sync
date: 2018-12-07 19:03:56
categories:
- Golang
tags:
- Golang
- condition variable
- lock
_ sync map
- mutex
- sync pool
- rwmutex
- once
- waitgroup
comments: true
---

## 简介

尽管goroutine之间可以通过channel进行同步，sync包还提供了goroutine之间进行同步的其他方式，**sync包中的对象不要进行复制**

## Locker接口

```Go
type Locker interface {
    Lock()
    Unlock()
}
```

定义了Lock和Unlock方法，表示一个可以被加锁和解锁的对象

## 常见对象

* Mutex

提供了互斥锁的功能，默认处于解锁状态
对未加锁的Mutex进行解锁会panic

实现机制:

```Go
const (
    mutexLocked = 1 << iota // 通过mutex.state & mutexLocked确定锁是否处于加锁状态
    mutexWoken // 通过mutex.state & mutexWoken确定锁是否处于唤醒状态
    mutexStarving // 通过mutex.state & mutexStarving确定锁是否处于饥饿状态
    mutexWaiterShift = iota //通过mutex.state >> mutexWaiterShift获取等待当前锁的goroutine数量


    // 为保证公平，mutex可以处于normal和starvation两种模式
    //
    // 在normal模式中，等待锁的各个goroutine位于FIFO的队列中，以FIFO的顺序来获取锁
    // 当处于队头的goroutine被唤醒以获取mutex时，如果此时一个处于running状态goroutine
    // 也在获取该mutex，并且由于该goroutine处于运行状态，所以很大概率处于队头的goroutine
    // 不会获取到mutex。因此，当对于等待队列的队头goroutine在等待mutex超过starvationThresholdNs
    // 纳秒后，mutex会进入starvation模式
    //
    // 在starvation模式中，mutex的控制权直接交给等待队列队头的goroutine，处于运行状态的goroutine
    // 不会尝试获取mutex的控制权，并且也不会尝试自旋操作，而是直接将自己放到等待队列尾部
    //
    // 当goroutine获取到锁的时间小于starvationThresholdNs纳秒或者该goroutine是等待队列中的最后一个
    // 元素时，mutex恢复到normal模式
    //
    // normal模式通常性能更好，而starvation则可以解决处于尾部的goroutine一直获取不到mutex的情况
    // tail latency问题

    // 切换到starvation模式需要等待的纳秒数，即1ms
    starvationThresholdNs = 1e6
)
type Mutex struct {
    state int32 // 锁的状态
    sema  uint32 // 信号量
}

func (m *Mutex) Lock() { // 尝试加锁，如果已经被锁定，则阻塞直到锁可以被加锁
    // 使用atomic包提供的CAS原子操作，将比较和交换看做一个整体 -> 对应的是X86上面特定的指令集(多核机器上还要对总线加锁以保证原子性)
    // 如果当前mutex处于解锁状态，则将其设置为加锁状态并直接返回
    if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
        if race.Enabled {
            race.Acquire(unsafe.Pointer(m))
        }
        return
    }

    var waitStartTime int64 // 当前goroutine等待mutex的时间
    starving := false       // mutex是否处于starvation模式
    awoke := false          // 当前goroutine是否被唤醒
    iter := 0               // 自旋操作的迭代此时(阻塞: 不占用CPU时间; 自旋: 占用CPU时间)
    old := m.state          // 当前mutex的状态
    for {
        // 在starvation模式下面不要尝试自旋操作，因为starvation模式下mutex的控制权直接交给等待队列头部的goroutine
        if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) { // mutex不处于starvation状态且可以进行自旋操作
            // 此时可以进行自旋操作，并且如果当前goroutine没有被唤醒的话，则将state设置为唤醒状态
            // 这样其他goroutine的Unlock操作将不再会唤醒等待mutex而阻塞的goroutine
            if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
                atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
                awoke = true
            }
            //进行自旋操作
            runtime_doSpin()
            iter++ // 自旋此时+1
            old = m.state
            continue // 继续进行自旋操作，直到runtime_canSpin不满足条件
        }
        // 如果不能进行自旋操作，则确定mutex要更新到什么状态
        new := old
        if old&mutexStarving == 0 { // 如果mutex没有处于starvation状态，将new第一位设置为加锁状态
            new |= mutexLocked
        }
        if old&(mutexLocked|mutexStarving) != 0 { // 如果mutex处于加锁或starvation状态，则new中goroutine的等待数量加1
            new += 1 << mutexWaiterShift
        }
        if starving && old&mutexLocked != 0 { // 如果mutex已经加锁且处于starvation状态，则new设置为starvation状态
            new |= mutexStarving
        }
        if awoke {
            if new&mutexWoken == 0 { // 如果goroutine已经被唤醒但是mutex不在唤醒状态，则抛出错误
                throw("sync: inconsistent mutex state")
            }
            new &^= mutexWoken // 取消new的唤醒状态
        }
        if atomic.CompareAndSwapInt32(&m.state, old, new) { // 将mutex的状态由old更改为new
            if old&(mutexLocked|mutexStarving) == 0 {
                break // 如果mutex未锁定且未处于starvation状态，则当前goroutine获取到锁，退出循环
            }
            queueLifo := waitStartTime != 0 // 判断当前goroutine是否已经在mutex的等待队列中
            if waitStartTime == 0 {
                waitStartTime = runtime_nanotime() // 设置goroutine等待mutex的开始时间
            }
            runtime_SemacquireMutex(&m.sema, queueLifo) // 如果queueLifo=true，则将当前goroutine插队到等待队列头部，此处通过信号量操作进行阻塞
            starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs // 判断是否需要加mutex设置为starvation状态
            old = m.state
            if old&mutexStarving != 0 { // 如果mutex已处于starvation状态
                if old&(mutexLocked|mutexWoken) != 0 || old>>mutexWaiterShift == 0 {
                    throw("sync: inconsistent mutex state")
                }
                delta := int32(mutexLocked - 1<<mutexWaiterShift) // 加锁、唤醒和starvation模式
                if !starving || old>>mutexWaiterShift == 1 { // 如果不需要设置starvation模式或者只剩下一个等待的goroutine，则取消mutex的starvation状态
                    delta -= mutexStarving
                }
                atomic.AddInt32(&m.state, delta) // 设置mutex状态
                break // 当前goroutine获取到mutex，跳出循环
            }
            awoke = true // 未处于starvation模式，继续循环
            iter = 0
        } else {
            old = m.state //CSA操作失败，将old设置为mutex状态，继续进行循环
        }
    }

    if race.Enabled {
        race.Acquire(unsafe.Pointer(m))
    }
}

func (m *Mutex) Unlock() {
    if race.Enabled {
        _ = m.state
        race.Release(unsafe.Pointer(m))
    }

    new := atomic.AddInt32(&m.state, -mutexLocked)
    if (new+mutexLocked)&mutexLocked == 0 { // 不要对没有加锁的mutex进行解锁操作
        throw("sync: unlock of unlocked mutex")
    }
    if new&mutexStarving == 0 { // mutex没有处于starvation状态
        old := new
        for {
            // 如果等待队列为空或者mutex已处于唤醒、加锁或starvation状态，则不再需要唤醒下一个等待的goroutine
            if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken|mutexStarving) != 0 {
                return
            }
            // 等待者数量减1，并设置mutex为唤醒状态，同时唤醒下一个等待的goroutine
            new = (old - 1<<mutexWaiterShift) | mutexWoken
            if atomic.CompareAndSwapInt32(&m.state, old, new) {
                runtime_Semrelease(&m.sema, false) // 通过信号量操作唤醒等待的goroutine
                return
            }
            old = m.state // 如果CAS操作失败，则继续循环操作
        }
    } else {
        // 在starvation模式下，直接唤醒等待队列中下一个goroutine，将mutex控制权交给他
        // 此时mutex未设置解锁状态，下一个唤醒的goroutine将会设置该状态
        runtime_Semrelease(&m.sema, true)
    }
}
```

* RWMutex

```Go
func (rw *RWMutex) Lock() // 写锁
func (rw *RWMutex) Unlock()

func (rw *RWMutex) RLock() // 读锁
func (rw *RWMutex) RUnlock()
```

提供了读写锁的功能，适用于大量读取、少量写入的情况
只有一个goroutine可以得到写锁; 可以有任意多个goroutine可以得到读锁
读锁和写锁之间互斥

* WaitGroup

```Go
func (wg *WaitGroup) Add(delta int)
func (wg *WaitGroup) Done()
func (wg *WaitGroup) Wait()
```

用于等待一组goroutine的结束

* Cond

```Go
func NewCond(l Locker) *Cond
func (c *Cond) Broadcast()
func (c *Cond) Signal()
func (c *Cond) Wait()
```

条件变量，用于goroutine之间通过事件通知来进行通信

* Pool

```Go
func (p *Pool) Get() interface{}
func (*Pool) Put
```

对象池，提高无状态的对象复用程度，降低GC压力
可以同时被多个协程使用
当池中对象仅有池中的唯一引用时，对象可能会被GC掉

* Once

```Go
func (o *Once) Do(f func())
```

使函数只执行一次，主要用于初始化操作

* Map

```Go
func (m *Map) Delete(key interface{})
func (m *Map) Load(key interface{}) (value interface{}, ok bool)
func (m *Map) LoadOrStore(key, value interface{}) (actual interface{}, loaded bool)
func (m *Map) Range(f func(key, value interface{}) bool)
func (m *Map) Store(key, value interface{})
```

协程安全的Map