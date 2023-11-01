# go mod

## 创建一个 go.mod
1. 初始化存在代码的文件夹
```
cd $GOPATH
mkdir greetings && cd greetings
```
2. 初始化go.mod `go mod init demo.com/alex/greetings`
3. 写一个go文件 [greetings.go](greetings.go)

注意点：
- 申明package
- 函数首字母必须大写，否则无法被外部调用
```golang
package greetings

import "fmt"

// Hello 注意函数首字母大写，否则不可被外部调用
func Hello(name string) string {
	message := fmt.Sprintf("Hi, %v, welcome", name)
	return message
}
```

## 调用包中的方法试试

1. 编辑 [hello.go](../demo/hello.go)
```go
package main

import (
	"demo.com/alex/greetings"
	"fmt"
)

func main() {
	message := greetings.Hello("我爱罗")
	fmt.Println(message)
}
```

2. 用 `go mod edit` 编辑demo.com/alex/greetings模块

因为这个包还没有上传到外部仓库，如 `github.com`，因此我们要将modules路径重定向到本地路径，命令如下：
```
go mod edit --replace demo.com/alex/greetings=../greetings
```
3. 接下来加载这个包 `go mod tidy`
4. 运行一下试试 `go run hello.go`