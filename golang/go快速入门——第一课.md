# go 快速入门

帮助文档：
> [Golang标准库文档](https://studygolang.com/pkgdoc)
> 
> [如何使用go语言编程](https://go-zh.org/doc/code.html)
> 
> 官网：https://golang.google.cn/ 

## 1. 安装golang

## 2. 设置 GOPROXY 

因为墙的存在，go get从国外服务器下载资源经常会失败，因此启用国内镜像非常有必要，我这里选择 [七牛云](https://goproxy.cn/)

临时修改
```
export GO111MODULE=on
export GOPROXY=https://goproxy/cn,direct
```

修改后始终有效
```
echo "export GO111MODULE=on" >> ~/.profile
echo "export GOPROXY=https://goproxy.cn" >> ~/.profile
source ~/.profile
```

也可以用 https://goproxy.io/zh/ 一个全球代理

### 配置Goland的modules，解决无法显示依赖包的问题
配置路径在 perferences -> Go -> Go Modules

设置GOPROXY=direct

![](../assets/16288472977494.jpg)

## 3. go quickstart

1. 创建一个项目文件夹，存放你的go源码

> 注意 工程项目必须在 GOPATH 工作目录下
```
cd $GOPATH
mkdir demo
cd demo
```

2. 为你的代码开启依赖项跟踪
```
go mod init github.com/alex/demo
```

3. 设置module重定向 `go mod edit --replace github.com/alex/demo=../demo`

因为代码并没有上传到github.com服务器上，因此需要重定向到本地

## 4. 调用外包中的代码
1. 找一个想要使用的外部包 rsc.io/quote
2. `import` 导入这个包，如下
```golang
package main

import "fmt"
import "rsc.io/quote"

func main() {
    fmt.Println(quote.Go())
}
```
3. 加载这个包到mod中 `go mod tidy`

执行过程提示：
```
go: finding module for package rsc.io/quote
go: found rsc.io/quote in rsc.io/quote v1.5.2
```

4. 运行你的代码 `go run test.go`


> [Uber Go 语言编码规范](https://github.com/xxjwxc/uber_go_guide_cn)


golang 工程项目注意点：

- 同级目录下的go文件属于同一个package
- 调用外部（不是同一个package）的函数或方法的首字母必须大写
- 同级目录下的其他go文件的函数可以直接调用，无需import

