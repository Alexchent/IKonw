# HTTP常见错误

- 301 永久重定向 
- 302 暂时重定向
- 401 没有登录
- 403 没有权限
- 404 请求的资源不存在
- 405 禁用请求中指定的方法
- 427 too many request 请求被限流
- 499 client has closed connection 服务端处理时间过长，客户端等待超时主动关闭了连接
- 500 服务端程序发生异常
- 501 尚未实施 服务器不具备完成请求的功能
- 502 bad gateway
- 503 服务暂时不可用
- 504 gateway timeout
- 505 http版本不支持，服务器不支持请求中所用的http协议版本

## 499
服务处理处理时间过长，客户端主动关闭了连接。
如果两次post请求提交过快，也会出现499，nginx认为这是不安全的。可以修改客户端调用流程，收到post请求返回结果后，立即调用一个查询接口，这样就可以避免重复提交。

## 502 bad gateway
502服务器作为网关或代理时，下一家的web服务器返回了非法的应答。nginx 502我们可以通过查看具体的错误日志来排查

1. nginx与web服务器不通，可能是反向代理配置错误，或防火墙问题
2. php-cgi没有空闲worker处理请求；php执行时间过长；或者php-cgi进程死掉

下面是几个例子：
### fastcgi缓存区设置过小
`2013/01/17 13:33:47 [error] 15421#0: *16 upstream sent too big header while reading response header from upstream`

修改nginx配置，提高缓存区大小
```
http {
    ...
    fastcgi_buffers 8 16k
    fastcgi_buffer_size 32k
    ...
}
```

### 代理缓冲区设置过小
如果使用nginx反向代理，如果header过大，超过默认的1k，就会引发 upstream sent too big header。
```
server {
    listen 80
    server_name *.lxy.me
    location / {
        ...
        proxy_buffer_size 64;
        proxy_buffers 32 32k;
        proxy_busy_buffers_size 128k;
        
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxt_set_header X-Forward-For $proxy_add_x_forward_for;
    }
}
``` 

### 默认php-fpm创建的worker进程不足
php-cgi处理请求时同步阻塞的，当没有空闲的worker进程时，就会出现502

### php执行超时

### nginx等待时间超时
处理请求的程序超过了nginx的等待时间，也会出现502


## 503 Service Unavaliable
表示服务器处于不可接受请求的状态

通常的原因是服务器处理停机维护或以超载。

nginx 503 通常是因为后端服务器(例如php-fpm)无法响应请求导致的，可能的原因如下
### 后端服务器故障，nginx无法连接到该服务器
### 连接池耗尽
### 后端服务器过载

上面是比较常见的情况，但不完全，具体还需要查看nginx错误日志来定位

### 504 Gateway Timeout
表示扮演网关或者代理服务器无法在规定时间内获得想要的响应。

通常是由于被代理的web服务处理超时导致的，需要通过nginx错误日志来定位具体原因。

> https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/503