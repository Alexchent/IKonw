# go 学习资料

- [7 个优质开源的 Go 项目](https://juejin.cn/post/7092788846781267975)
- [go语言101](https://gfw.go101.org/article/101.html)
- [go编程之旅](https://golang2.eddycjy.com/)
- [go语言原本](https://golang.design/under-the-hood/)


## go 常用指令


初始化mod
```
go mod init appName
```

拉去mod包
```
go mod tidy
```

安装go可执行程序
```
go install
```

运行go文件
```
go run main.go
```

编译go程序
```
go build main.go


# 显示编译过程
go build -gcflags=-S main.go
```