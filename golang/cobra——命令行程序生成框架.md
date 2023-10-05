# cobra

> [cobra-cli](https://github.com/spf13/cobra-cli/blob/main/README.md)


### 第一步安装 cobra-cli
```bash
go install github.com/spf13/cobra-cli@latest
```

### 第二步 创建一个cobra 项目

1. 在一个空项目中添加cobra

初始化一个module
```
cd $HOME/code
mkdir myapp
go mod init github.com/username/myapp
```
开始使用cobra-cli构建应用程序
`cobra-cli init` 会初始化一个新的cobra项目
```bash
cd $HOME/code/myapp
cobra-cli init
go run main.go -h
```

2. 或者直接创建一个cobra应用
```
cobra-cli create appName
```

