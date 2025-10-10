# Redis Stack

## 安装

### 安装在 Docker 上

> https://redis.io/docs/stack/get-started/install/docker/

生产环境推荐 `redis/redis-stack-server`
```
docker run -d --name redis-stack-server -p 6379:6379 redis/redis-stack-server:latest
```

开发环境
```
docker run -d --name redis-stack -p 6379:6379 -p 8001:8001 redis/redis-stack:latest
```

现在就可以通过 http://localhost:8001 访问了