# go的内存分配

Go的内存配置算法基于TCMalloc（Thread Cache Malloc线程缓存分配器）内存分配算法实现的，go的内存分配器核心实现在内置运行时（runtime）

## 核心思想
go程序在启动时，会分配一块连续内存（注意这是还只是虚拟内存，没有实际分配内存），切成小块后自行管理

1. 每次从操作系统申请一大块内存，以减少系统调用
2. 将申请到的大块内存按预算的设定切分成小块，构成链表
3. 为对象分配内存的时候，只需要从大小合适的链表中取出一小块
4. 回收对象内存时，将该小块内存重新归还到原链表
5. 如果闲置的内存过多，则会归还部分给操作系统

## 分配流程
GO内存分配器分配对象时，根据对象的大小，分成三类：小对象（<=16B），一般对象（16B到32KB之间），大对象（>32KB）

- 32KB的对象直接从mheap上分配
- <=16B的对象使用cache的tiny分配器分配
- （16B，32KB）的对象，首先计算对象规格大小，然后使用mcache上相应规格大小的mspan分配
- 如果mcache上没有相应规格大小的mspan，则向mcentral申请
- 如果mcentral也没有则向mheap申请
- 如果还是没有则向操作系统申请


## 内存管理组件
内存管理器由**mcache**, **mcentral**, **mheap**3种组件构成： 三级管理结构是为了方便对span进行管理，加速对span对象的访问和分配，这三个结构在runtime中分别有对应的mcache.go、mcentral.go、mheap.go文件。

> mcache：保存的是各种大小的Span，并按Span class分类，小对象直接从mcache分配内存，它起到了缓存的作用，并且可以**无锁访问** Go中是**每个P拥有1个mcache**，因为在Go程序中，当前最多有GOMAXPROCS个线程在运行，所以最多需要GOMAXPROCS个mcache就可以保证各线程对mcache的无锁访问

> mcentral：是所有线程共享的缓存，需要加锁访问，它按Span class对Span分类，串联成链表，当mcache的某个级别Span的内存被分配光时，它会向mcentral申请1个当前级别的Span

> mheap：是堆内存的抽象，把从OS（系统）申请出的内存页组织成Span，并保存起来。当mcentral的Span不够用时会向mheap申请，mheap的Span不够用时会向OS申请，向OS的内存申请是按页来的，然后把申请来的内存页生成Span组织起来，同样也是需要加锁访问的。 mheap主要用于大对象的内存分配，以及管理未切割的mspan，用于给mcentral切割成小对象

> [GO内存管理和分配策略](https://cloud.tencent.com/developer/article/2260865?areaId=106001)