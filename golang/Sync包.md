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

