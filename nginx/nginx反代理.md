# nginx 反向代理

server 模块中配置 **proxy_pass**
```
server {
    listen 80;
    server_name localhost;
    
    location / {
        root html;
        index index.html;
        proxy_pass 23.12.11.234:8080;
    }
}
```

# nginx 负载均衡

配置upstream模块+反向代理

内置策略由：轮询、权重轮询和ip_hash
扩展策略：url_hash、fair等等

## 轮询
按请求顺序，逐一分配到不用web服务器
```
upstream server_list {
    server localhost:8080;
    server localhost:8081 max_fails=3 fail_timeout=20s;
}

server {
    listen 80;
    server_name localhost;
    
    location / {
        root html;
        index index.html;
        proxy_pass http://server_list;
    }
}
```
参数说明：
fail_timeout	与max_fails结合使用

| 参数 | 作用 |
|---|---|
| max_fails |设置在fail_timeout参数设置的时间内最大失败次数，如果在这个时间内，所有针对该服务器的请求都失败了，那么认为该服务器会被认为是停机了，  |
| fail_time | 服务器会被认为停机的时间长度,默认为10s。 |
| backup | 标记该服务器为备用服务器。当主服务器停止时，请求会被发送到它这里。 |
| down | 标记服务器永久停机了 |

## 加权轮询
权重高的web服务器获得更多的流量，用于web服务器性能不均的情况
可以与**ip_hash**和**least_conn**配合使用
```
upstream server_list {
    server localhost:8080 weight=2;
    server localhost:8081 weight=1;
}
```

## ip_hash
每个请求按访问ip的hash值分配
```
upstream server_list {
    ip_hash;
    server localhost:8080;
    server localhost:8081;
}
```

## 最少连接数
web请求会被转发到连接数最少的服务器上
```
upstream server_list {
    least_conn;
    server localhost:8080;
    server localhost:8081;
}
```

## fair 按响应时间分配
需要安装第三方插件
```
upstream server_list {
    fair;    #实现响应时间短的优先分配
    server localhost:8080;
    server localhost:8081; 
}
```

## url_hash
需要安装第三方插件
```
upstream server_list {
    hash $request_uri;    #实现响应时间短的优先分配
    server localhost:8080;
    server localhost:8081; 
}
```