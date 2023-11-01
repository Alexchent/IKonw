# nginx配置

## 概述
nginx的配置文件`conf/nginx.conf`按以下结构组织：

| 配置块 | 功能描述 |
|---|---|
| 全局块 | 与nginx运行相关的设置 |
| events | 与网络连接相关的设置 |
| http | 代理、缓存、日志、虚拟主机等配置 |
| server | 虚拟主机的参数设置，一个http可以有多个server |
| location| 定义请求路由以及处理方式 |

![](../assets/nginx.conf.awebp)

下面是一个nginx.conf的样本
```
# 全局段配置
# ------------------------------

# 指定运行nginx的用户或用户组，默认为nobody。
#user administrator administrators;

# 设置工作进程数，通常设置为等于CPU核心数。
#worker_processes 2;

# 指定nginx进程的PID文件存放位置。
#pid /nginx/pid/nginx.pid;

# 指定错误日志的存放路径和日志级别。
error_log log/error.log debug;

# events段配置信息
# ------------------------------
events {
    # 设置网络连接序列化，用于防止多个进程同时接受到新连接的情况，这种情况称为"惊群"。
    accept_mutex on;

    # 设置一个进程是否可以同时接受多个新连接。
    multi_accept on;

    # 设置工作进程的最大连接数。
    worker_connections  1024;
}

# http配置段，用于配置HTTP服务器的参数。
# ------------------------------
http {
    # 包含文件扩展名与MIME类型的映射。
    include       mime.types;

    # 设置默认的MIME类型。
    default_type  application/octet-stream;

    # 定义日志格式。
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for';

    # 指定访问日志的存放路径和使用的格式。
    access_log log/access.log myFormat;

    # 允许使用sendfile方式传输文件。
    sendfile on;

    # 限制每次调用sendfile传输的数据量。
    sendfile_max_chunk 100k;

    # 设置连接的保持时间。
    keepalive_timeout 65;

    # 定义一个上游服务器组。
    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #此服务器为备份服务器。
    }

    # 定义错误页面的重定向地址。
    error_page 404 https://www.baidu.com;

    # 定义一个虚拟主机。
    server {
        # 设置单个连接上的最大请求次数。
        keepalive_requests 120;

        # 设置监听的端口和地址。
        listen       4545;
        server_name  127.0.0.1;

        # 定义location块，用于匹配特定的请求URI。
        location  ~*^.+$ {
           # 设置请求的根目录。
           #root path;

           # 设置默认页面。
           #index vv.txt;

           # 将请求转发到上游服务器组。反向代理
           proxy_pass  http://mysvr;

           # 定义访问控制规则。
           deny 127.0.0.1;
           allow 172.18.5.54;          
        } 
    }
}
```
## 全局配置
全局配置中我们重点要关注的是
1. user user [group] # 允许运行nginx的用户|组
2. worker process   # 设置工作进程数，通常设置为等于CPU核心数

> 如果您的web项目有生成文件的操作，则文件的用户即为这里指定的用户。如果同时您还有一个有`supervisor`启动的服务，要操作同一个文件，但supervisor指定的用户与nginx不同，权限不一致，将出现问题。

## events
- **worker_connections** number；

每个worker process 同时开启的最大连接数

## http 
- access_log
- keepalive 配置连接超时时间
- keepalive_request number 限制单次连接请求次数

### server
每个server指定一个虚拟主机

#### location
每个location对应一个路由

##### 动静分离
1. 直接为静态文件设置一个别名或根目录
```
location ~* .(jpg|jpeg|png|gif|ico|css|js)$ {
    root /path/to/static/files;
    expires 30d;  # 设置缓存时间
}
```
1. 使用alias别名
```
location /static/ {
    alias /path/to/static/files/;
}
```

## 静态资源的优化
为了提高静态资源的传输效率，Nginx提供了以下三个主要的优化指令
- sendfile on # 开启高效传输模式，减少文件拷贝次数
- tcp_nopush  # 提高网络数据包的传输效率，
- tcp_nodelay
- gzip on     # 开启文本压缩

sendfile 配置文件 `http`、`server`、`location`

tcp_nopush 的工作原理是设置一个缓冲区，当缓冲区满时才进行数据发送，这样可以大大减少网络开销。

nginx gizp的好处和缺点
开启对文本内容的压缩，如css、js、xml、html可以减少带宽的消耗，提高响应速度，但是会消耗一定cpu资源

> gzip压缩与sendfile共存，设置 gzip_static on


## 跨域

跨域资源共享CORS是一种安全策略，用于控制哪些网站可以访问您的资源。当前端应用程序和后端API位于不同服务器上，通常会遇到跨域问题。nginx可以设置响应头来解决这个问题。

```
location / {
    # 其他配置...

    # 设置允许来自所有域名请求。如果需要指定域名，将'*'替换为您的域名。
    add_header 'Access-Control-Allow-Origin' '*';

    # 允许的请求方法。
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';

    # 允许的请求头。
    add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';

    # 允许浏览器缓存预检请求的结果，单位为秒。
    add_header 'Access-Control-Max-Age' 1728000;

    # 允许浏览器在实际请求中携带用户凭证。
    add_header 'Access-Control-Allow-Credentials' 'true';

    # 设置响应类型为JSON。
    add_header 'Content-Type' 'application/json charset=UTF-8';

    # 针对OPTIONS请求单独处理，因为预检请求使用OPTIONS方法。
    if ($request_method = 'OPTIONS') {
        return 204;
    }
}
```

生成环境，`access-control-allow-origin` 建议设置为指定的域名

## 防盗链

防盗链是指防止其他网站直接连接到你的网站资源，消耗你的带宽。nginx提供了非常方便的模块 `ngx_http_referer_module` 实现防盗链功能

```
location ~ .*.(gif|jpg|jpeg|png|bmp|swf)$ {
    valid_referers none blocked www.example.com example.com *.example.net;
    
    if ($invalid_referer) {
        return 403;
    }
}
```
上述的配置
valid_referers定义了合法的来源页面。**none**表示直接访问，**blocked**表示没有Referer头的访问，www.example.com和example.com是合法的来源域名，*.example.net表示example.net的所有子域名都是合法的来源。

$invalid_referer变量会在来源不在valid_referers列表中时变为"true"。

如果来源不合法，服务器将返回403禁止访问的状态码


注意事项：
- 设置防盗链可能影响搜索引擎的爬虫
- 如果你的网站使用CDN，确保CDN服务器也在`vaild_referers`列表中


> [Nginx配置指南](https://juejin.cn/post/7267003603095879714)
> 
> [Nginx 高性能优化配置](https://cloud.tencent.com/developer/article/1982125)