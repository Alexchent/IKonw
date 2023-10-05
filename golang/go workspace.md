# go workspace

工作区模式 `go.work`，取代以前的 `go replace`

## 引用一个本地包 demo.com/alex/greetings
 因为这个包还没上传到服务器 demo.com/alex 上，需要使用本地环境。
 
 假设你有一个在开发的包 `demo.com/alex/hello`, 需要`demo.com/alex/greetings` 包
 
###  执行以下操作，初始化 `hello` 包
 ```
 cd $GOHOME/src
 mkdir hello
 cd hello
 go mod init demo.com/alex/hello
 ```
###  同样在初始化一个 `greeting` 包
 ```
 cd $GOHOME/src
 mkdir greeting
 cd greeting
 go mod init demo.com/alex/greeting
 ```
### 进入 greeting 编辑一个 `go` 文件 
```golang
package greeting

import "fmt"

func Hello() {
    fmt.Println("hello world")
}
```

### 进入 hello 编辑一个 `main.go` 文件，引用greeting包中的函数
```golang
package main

import "demo.com/alex/greetings"

func main () {
    greetings.Hello()    
}
```

此时的目录结构如下：
```
├── greeting
│   ├── go.mod
│   └── main.go
└── hello
    ├── go.mod
    └── main.go
``` 

此时的程序还无法执行，因为 go mod 此时只会服务远程mod包，而greeting包还没有上传到远程服务器。

 
- 低于 1.18 版本，在 hello 目录下执行 `go replace demo.com/alex/greetings=../greetings`

看一下go.mod文件
```go.mod
module example/alex/hello

go 1.18

replace example/alex/greeting => ../greeting

require example/alex/greeting v0.0.0-00010101000000-000000000000
```
- 1.18 及之后的版本，在 workspace 目录下 执行 `go work init ./greetings`

此时的目录结构如下：
```
├── go.work
├── greeting
│   ├── go.mod
│   └── main.go
└── hello
    ├── go.mod
    └── main.go
```

go.work
```
go 1.18

use (
	./greeting
	./hello
)
```




