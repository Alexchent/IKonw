# 终止goroutine的方式

## 通过channel传递关闭信号
```go
package main

func main() {
    quit := make([]struct{})
    go func() {
		defer fmt.Println("goroutine exit")
		for {
			select {
			case <-quit:
				fmt.Println("quit")
				return
			default:
			   fmt.Println("Timer fired")
			}
		}
	}()
	
	time.Sleep(3 * time.Second)
	quit <- struct{}{}
	fmt.Println("close")
}

```

## 关闭监听的channel
### select 接收
```go
package main

func main() {
   timer := time.NewTicker(1 * time.Second)
	defer timer.Stop()

	quit := make(chan struct{})
	// 定时执行
	go func() {
		defer fmt.Println("Timer1 goroutine exit")
		for {
			select {
			case <-timer.C:
				fmt.Println("Timer1 fired", time.Now().Unix())
			case <-quit:
				fmt.Println("Timer1 quit")
				return
			}
		}
	}()
	go func() {
		defer fmt.Println("Timer2 goroutine exit")
		for {
			select {
			case <-timer.C:
				fmt.Println("Timer2 fired", time.Now().Unix())
			case <-quit:
				fmt.Println("Timer2 quit")
				return
			}
		}
	}()

   time.Sleep(3 * time.Second)
   close(quit)
   time.Sleep(5 * time.Second)
}
```

### for range 接收
```go
package main

func main() {
   c := make(chan int)
	for i := 0; i < 2; i++ {
		i := i
		go func() {
			defer fmt.Println("worker", i, "exit")
			for v := range c {
				fmt.Printf("worker %d got %d\n", i, v)
				//time.Sleep(time.Millisecond * 500)
			}
		}()
	}
	// 每个50毫秒生成一个数据传入通道c内
	c <- 1
	c <- 2
	c <- 3
	time.Sleep(time.Second * 1)
	close(c)
	time.Sleep(time.Second * 1)
}
```

## context传递取消信号
```go

```