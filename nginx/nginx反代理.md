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
nginx是七层负载均衡，工作在应用层，可针对http应用实施一些分流策略。除此以外还有工作在四层的负载均衡[LVS](https://juejin.cn/post/6966411996589719583)

- 轮询
- 加权轮询
- ip_hash
- 最少连接数
- fair最少响应时间 （第三方）
- url_hash（第三方）

配置upstream模块+反向代理proxy_pass

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

# 负载均衡架构
负载均衡（Load Balance）是分布式系统架构设计中必须考虑的因素之一，它通常是指，将请求/数据【均匀】分摊到多个操作单元上执行，负载均衡的关键在于【均匀】。

1.【客户端层】到【反向代理层】的负载均衡，是通过“**DNS轮询**”实现的
2.【反向代理层】到【站点层】的负载均衡，是通过“**nginx**”实现的
3.【站点层】到【服务层】的负载均衡，是通过“**服务连接池**”实现的
4.【数据层】的负载均衡，要考虑“数据的均衡”与“请求的均衡”两个点，常见的方式有“按照范围水平切分”与“hash水平切分”

1. 范围分割，比如按时间范围分割

优点：规则简单，易扩展
缺点：可能负载不均衡，比如当前时段的请求量要高于历史数据

2. 对key进行hash散列，取模

优点: 规则简单，负载比较均衡
缺点：不易扩展，需要迁移数据

# 获取真是客户端ip
首先，一个请求肯定是可以分为请求头和请求体的，而我们客户端的IP地址信息一般都是存储在请求头里的。如果你的服务器有用Nginx做负载均衡的话，你需要在你的location里面配置`X-Real-IP`和`X-Forwarded-For`请求头

```
location ^~ /your-service/ {
    proxy_set_header        X-Real-IP       $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://localhost:60000/your-service/;
}
```

如果经过负载均衡，则$remote_addr只能获得上级服务器的ip地址，
$proxy_add_x_forwarded_for则存储则经过的所有服务ip地址，其中第一个就是客户端ip