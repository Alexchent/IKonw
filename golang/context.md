# context.Context

go在1.7引入了 `context`包，目的是**在不同grouine之间传递截止日期、取消信号，以及共享上下文信息**

在 Go 的日常开发中，`Context` 上下文对象无处不在，无论是处理网络请求、数据库操作还是调用 RPC 等场景下，都会使用到Context

## 作用
在 Goroutine 构成的树形结构中对**信号进行同步以减少计算资源的浪费是 context.Context 的最大作用**。
![](../assets/golang-context-usage.png)

每个context.Context都会从最顶层的goroutine一层一层传递到最下一层，可以上层goroutine执行失败时，及时将信号同步给下层，避免资源浪费。

### 控制子协程退出
context提供了一种机制，可以在多个goroutine之间进行通信和控制。使用context包能够有效的控制程序的并发性，提供程序的程序的健壮性和性能。



### 超时控制

在查询数据库、调用RPC服务、调用HTTP接口等场景，这些操作都是阻塞的，如果一直不返回，会影响用户体验。针对这些情况的解决方式通常是设置一个超时时间，超过后自动取消操作

### 传递上下文数据

这个作用在链路追踪中非常重要

## 创建context的方式
- context.Background()
- context.TODO()
- context.withValue()
- context.WithCancel()
- context.WithCancelCause() // 附加取消的原因
- context.WithDeadline()
- context.WithTimeout()

### context.Background()
`context.Background()` 函数返回一个非nil的Context，它没有携带任何的值，也没有取消和超时信号，通常作为根使用

```
ctx := context.Background()
```

### context.TODO()
它的返回结果和 `context.Background()` 函数一样，但是它们的使用场景是不一样的，如果不确定使用哪个上下文时，可以使用它。
```
ctx := context.TODO()
```

### context.WithValue()
接收一个父 Context 和一个键值对 key、val，返回一个新的子 Context，并在其中添加一个 key-value 数据对
```go
ctx := context.WithValue(parentCtx, "session-id:123", "userInfo:tony")
```

### context.WithCancel()
这个函数很常用，接收一个父 Context，返回一个新的子 Context 和一个取消函数，**当取消函数被调用时，子Context会被取消，同时会向子 Context 关联的 Done() 通道发送取消信号，届时其衍生的子孙 Context 都会被取消**。这个函数适用于手动取消操作的场景。

```go
ctx, cancel := context.Cancel(context.Background)
defer cancel()
```

### context.WithDeadline()
接收一个父 Context 和一个截止时间作为参数，返回一个新的子 Context。当截止时间到达时，子 Context 其衍生的子孙 Context 会被自动取消。这个函数适用于需要在特定时间点取消操作的场景。
```go
deadline := time.Now().Add(time.Second * 2)
ctx, cancelFunc := context.WithTimeout(parentCtx, deadline)
defer cancelFunc()

```

### context.WithTimeout()
功能和`WithDeadline()`是一样的，低底层也是调用的`WithDeadline()`，区别是它传入的是一个时间，比如2s后超时
```
ctx, cancel := context.WithTimeout(parentCtx, 2 * time.Second)
```

> [一文掌握go并发模式context上下文](https://juejin.cn/post/7233981178101186619)