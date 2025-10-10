# docker 部署mongodb

1. 拉取镜像
`docker pull mongodb:4`



1. 启动容器

```
docker run -d \
--name mongodb \
-p 27017:27017 \
-v ~/docker/mongodb/datadb:/data/db \
-e MONGO_INITDB_ROOT_USERNAME=admin \
-e MONGO_INITDB_ROOT_PASSWORD=admin \
--privileged=true \
--restart always \
mongo:4
```

参数说明

    -d 后台运行容器
    --name mongodb 运行容器名
    -p 27017:27017 将容器的27017端口映射到主机的27017端口
    -v /mydata/mongodb/datadb:/data/db 文件挂载目录
    -e MONGO_INITDB_ROOT_USERNAME=admin 指定用户名
    -e MONGO_INITDB_ROOT_PASSWORD=admin 指定密码
    --privileged=true 使得容器内的root拥有真正的root权限
    --restart always 跟随docker一起启动，即docker启动时会自动运行容器