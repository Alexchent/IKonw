# go内存泄露

认为造成的内存泄露，8成以上都是goroutine泄露

## 造成泄露的原因
1. goroutine陷入死循环
2. 无缓冲或缓存已满的channel只有写，没有读
3. 无缓冲或缓存已空的channel只有读，没有写
4. select的case上的channel操作始终阻塞

## 检测
自带的检测工具pprof，和uber开源的 [goleak](https://github.com/uber-go/goleak) 可以集成到单元测试中，可以快速检测goroutine泄露。


安装goleak `go get -u go.uber.org/goleak`