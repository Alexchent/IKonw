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