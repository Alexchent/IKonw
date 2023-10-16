# go context

go在1.7引入了 `context`包，目的是**在不同grouine之间或跨API边界传递超时、取消信号，以及共享上下文信息**

在 Go 的日常开发中，`Context` 上下文对象无处不在，无论是处理网络请求、数据库操作还是调用 RPC 等场景下，都会使用到 Context

## 创建context的方式
- context.Background()
- context.TODO()
- context.Value()
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

> [一文掌握go并发模式context上下文](https://juejin.cn/post/7233981178101186619)