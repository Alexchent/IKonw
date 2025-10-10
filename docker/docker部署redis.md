# docker 部署redis-cli

### 部署

#### 用于开发环境
启动 `redis-stack` 用于开发环境，访问 IP:8001 是官方提供的图形管理工具
```
docker run -d --name redis-stack -p 6379:6379 -p 8001:8001 redis/redis-stack:latest
```
#### 用于生产环境
启动 `redis-stack-server` 不含图形管理工具
```
docker run -d --name redis-stack-server -p 6379:6379 redis/redis-stack-server:latest
```

### redis-cli 连接
```
docker exec -it redis-stack redis-cli
```


更多内容请参考官网 
> [Run Redis Stack on Docker](https://redis.io/docs/getting-started/install-stack/docker/)