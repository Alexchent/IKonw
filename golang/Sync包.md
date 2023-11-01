# Sync包 

## WaitGroup 同步等待组

- `func (wg *WaitGroup) Add(delta int)`

Add这个方法，用来设置到WaitGroup的计数器的值。我们可以理解为每个waitgroup中都有一个计数器, 用来表示这个同步等待组中要执行的goroutin的数量

- `func (wg *WaitGroup) Done()`

Done()方法，就是当WaitGroup同步等待组中的某个goroutine执行完毕后，设置这个WaitGroup的counter数值减1

- `func (wg *WaitGroup) Wait()`

Wait()方法，表示让当前的goroutine等待，进入阻塞状态。一直到WaitGroup的计数器为零。才能解除阻塞， 这个goroutine才能继续执行

## Mutex （互斥锁）
同时只有一个groutine能获得锁

加锁
```
func (m *Mutex) Lock()
```
解锁
```
func (m *Mutex) Unlock()
```
## RWMutex（读写锁）
特性：
1. 同时只有一个groutine能获的写锁
2. 同时可以有多个groutine获得读锁
3. 读锁和写锁不能同时存在

所以RWMutex可以同时加多个读锁，或一个写锁，常用与读次数远远高于写次数的场景

- 加写锁 `func (rw *RWMutex) Lock()`
- 解写索 `func (rw *RWMutex) Unlock()`
- 加读锁 `func (rw *RWMutex) RLock()`
- 解读锁 `func (rw *RWMutex) RUnlock()`

## Once
`sync.Once` 可以保证 GO程序运行期间，某个代码片段只会执行一次，即使这个代码发生了painc
```go
func main() {
    o := &sync.Once{}
    for i := 0; i < 10; i++ {
        o.Do(func() {
            fmt.Println("only once")
        })
    }
}

$ go run main.go
only once
```

注意事项：
1. once内通过mutex加锁，所以Do内部重复调用Do会导致死锁
```go
func main() {
    o := sync.Once{}
    o.Do(func() {
        o.Do(func() {
            fmt.Println("init ...")
        })
    })
}
```
2. once内的代码执行过程中发生error，once无法感知，始终只会执行一次

### 原理
once的结构体包含两个字段done和互斥锁m
```go
type Once struct {  
    done uint32
    m    Mutex
}
```
3. 两次调用sync.Once.Do方法传入不同的函数，只会执行第一次传入的函数

`sync.Once.Do`是once对位唯一暴露的方法，该方法接收一个无参数的函数
- 如果改函数已经执行过，会直接返回
- 如果没有执行过，那么调用doSlow执行函数

doslow内部通过mutex进行并发控制，判断done==0，如果成立，执行函数，在通过`atomic.StoreUint32(&o.done, 1)`将done置为1

## Cond


常见的使用场景：
- 单例模式，确保全局只有一个实例化对象，避免重复创建资源
- 延迟初始化，在程序需要用到某个资源时，动态的初始化
- 只执行一次的操作

# mutex 有几种模式
正常模式和饥饿模式
### 正常模式
所有groutine按先进先出的顺序获取锁，被唤醒的goroutine和新请求锁的goroutine，通常新的goroutine更容易获得锁（持续占用CPU）

### 饥饿模式
所有尝试获取锁的goroutine，都进入队列中排队，新请求锁的goroutine不会进行锁获取（禁用自旋）

### 切换条件
当一个G等待时间超过**1ms**，进入饥饿模式；

队列中的所有任务完成，或队列中的G等待时间都低于1ms切回正常模式