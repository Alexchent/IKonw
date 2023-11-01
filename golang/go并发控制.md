# go并发控制

提到goroutine的并发控制，很多人都会想到锁。go提供了互斥锁mutex和读写锁RWMutex，除了锁以外还提供了原子操作sync.atomic等。但是这些机制关注点重点是并发安全。而我们本次讨论的是并发控制

go中并发控制有三种方式：sync.waitGroup、channel和context

## waitGroup
主要有如下几个方法
- Add()
- done()
- wait()

每多个goroutine都需要 `add(1)`，G结束后用done()销毁，相当于add(-1)。wait则会阻塞等待所有协程完成。

waitGroup 尤其适用于：某个任务需要多个goroutine协同，只有全部goroutine完成时，任务才算完成。因此同名称的含义一样，它是一种等待方式

如果需要主动通知一个goroutine结束，就无法满足要求了。
## channel
channel是官方推荐的并发控制模式
有两种形式的channel，缓冲模式和无缓冲模式

我们经常采用chanel+select的方式来进行并发控制

## context
context可以在多个goroutine之间传递超时、取消信号，还可以传递上下文信息

多个goroutine可以通过接收`ctx.done()`的信号，同时进行某些操作


