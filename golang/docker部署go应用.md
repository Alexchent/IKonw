# docker 部署golang应用
默认docker已经安装好了


## 1. 创建一个golang web程序 main.go

- `cd $GOPATH/src` 进入本地的go工作目录
- `mkdir gin` 创建一个目录保存golang文件
- `touch main.go` 创建一个golang文件 main.go 

main.go文件内容如下，这是一个demo，你可以自定义
```golang
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

var db = make(map[string]string)

func setupRouter() *gin.Engine {
	r.GET("/ping", func(c *gin.Context) {
		c.String(http.StatusOK, "pong")
	})

	r.GET("/user/:name", func(c *gin.Context) {
		user := c.Params.ByName("name")
		value, ok := db[user]
		if ok {
			c.JSON(http.StatusOK, gin.H{"user": user, "value": value})
		} else {
			c.JSON(http.StatusOK, gin.H{"user": user, "status": "no value"})
		}
	})

	return r
}

func main() {
	r := setupRouter()
	// Listen and Server in 0.0.0.0:8080
	r.Run(":8000")
}

```
    
## 2. 在main.go同级目录创建一个 `Dockerfile`文件

```
FROM golang:latest
MAINTAINER chentao
WORKDIR $GOPATH/src/blog
ADD . $GOPATH/src/blog
#RUN go build main.go
EXPOSE 8000
ENTRYPOINT ["./main"]
```

说明：

- `FROM: golang:latest` 依赖基础 golang:lastest 镜像
- `MAINTAINER chentao` 镜像维护者的名字
- `WORKDIR $GOPATH/src/gin` 工作目录
- `ADD . $GOPATH/src/gin` 将当前目录的所有文件复制到工作目录
- `RUN go build main.go` 镜像默认执行命令 `go build main.go`
- `EXPOSE 8000` 暴露端口号8000

### 特别注意：交叉编译

因为墙的原因 `RUN go build main.go` 可能会执行超时，此时我们可以选择直接在本地环境编译好在生产环境的可执行文件。因为开发环境与生产环境不一样，需要[交叉编译](https://golang.google.cn/doc/install/source)。

MAC下编译LINUX可执行程序 `GOOS=linux GOARCH=amd64 go build main.go`

## 3. 构建 go-web 镜像
```
docker build -t gin-web:v1 .
```

go-web是新建的镜像名，v1是版本号


## 4. 运行 go-web 镜像
```
docker run -it -p 8000:8000 --name my-go-web gin-web:v1 
```

现在，访问一下试试 http://127.0.0.1/ping

## 5. docker-compose 编排

Dockerfile同级目录下编写一个 docker-compose.yml
```yml
# 启动容器 docker-compose -f docker-compose.yml up -d
# 说明：
# -f docker-compose.yml 可忽略
# -d 你表示后台运行

version: "3.0"
services:
  # 生成的容器名：<顶级目录名> 此处是gin
  # 包含一个由Dockerfile构建image gin-blog
  blog:
    # 执行当前路径下的Dockerfile
    build: .
    # 绑定本机8000端口和容器的8000端口
    ports:
      - "8000:8000"
```

然后，通过docker.compose启动容器`docker-compose -f docker-compose.yml up`

![](assets/16421635164534.jpg)



 