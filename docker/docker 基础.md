# docker 基础

hub.docker.com

账号：chentao2019hh

密码：4503335poiQ

---

https://dashboard.daocloud.io/packages/explore?from=signup

账号：alexc
微信绑定

> [docker命令大全](https://www.runoob.com/docker/docker-command-manual.html)

## 1. 基本命令
```bash
docker pull   //获取image
docker build  //创建image
docker images //列出所有image
docker run    //创建并运行container

docker ps    //列出所有正在运行的container
docker ps -l //查看当前未运行的容器
docker ps -a //查看所有本地的容器

#启动/关闭/重启容器
docker start/stop/restart {container-name}

#进入容器内部
docker exec -it {container-id} /bin/bash

#删除容器
docker rm     //删除container
docker rmi    //删除image
docker cp     //在host和container之间拷贝文件
docker commit //保存改动为新的image

#查看容器信息
docker inspect container -name

```

### docker run 常用参数说明

- `-p` 指定端口映射，格式为：主机(宿主)端口:容器端口
- `-d` 后台运行容器，并返回容器ID；
- `-i` 以交互模式运行容器，通常与 -t 同时使用
- `-t` 为容器重新分配一个伪输入终端，通常与 -i 同时使用
- `-e username="ritchie"` 设置环境变量，
- `-v` 绑定一个卷
- `--name="nginx-lb"` 为容器指定一个名称
- `--link=[]` 添加链接到另一个容器

例子：
启动一个mysql
```
docker run --name docker-mysql -it -d \
-p 3306:3306 \
-v ./conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=123456 mysql
```

-p 3306:3306 ：映射容器服务的 3306 端口到宿主机的 3306 端口，外部主机可以直接通过 宿主机ip:3306 访问到 MySQL 的服务。

-v ./conf:/etc/mysql/conf.d 映射 本机的./conf目录 到容器的/etc/mysql/conf.d

-e MYSQL_ROOT_PASSWORD=123456：设置 MySQL 服务 root 用户的密码。

### 删除所有dangling数据卷（即无用的Volume，僵尸文件）
docker volume rm $(docker volume ls -qf dangling=true)

### 删除所有dangling镜像（即无tag的镜像）
docker rmi $(docker images | grep "^<none>" | awk "{print $3}")

### 删除所有关闭的容器
docker ps -a | grep Exit | cut -d ' ' -f 1 | xargs docker rm

## 2. Dockerfile
```Dockerfile
FROM ubuntu		//base image
MAINTAINER ctao		//作者
RUN 				//执行命令
ADD				//添加文件
COPY 				//拷贝文件
CMD 				//执行命令
EXPOSE				//暴露端口
WORKDIR 			//指定路径
ENV					//设定环境变量
ENTRYPOINT 		//容器入口
USER				//指定用户
VOLUME				//
```

`docker build` 使用Dockerfile创建镜像

常见参数
- `-f ` 执行要使用的dockefile路径
- `--tag`, `-t` 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签

使用当前目录的Dockerfile创建镜像，标签名：runoob/ubuntu:v1
`docker build -t runoob/ubuntu:v1 . `

使用URL github.com/creack/docker-firefox 的 Dockerfile 创建镜像。
`docker build github.com/creack/docker-firefox`

指定Dockerfile文件
`docker build -f ~/docker/laravel/Dockerfile .`

## 3. [docker-desktop开启Kubernetes](https://github.com/AliyunContainerService/k8s-for-docker-desktop)