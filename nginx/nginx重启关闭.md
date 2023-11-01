## nginx 服务器重启命令，关闭

#### 测试nginx配置文件是否正确
```
nginx -t -c /path/to/nginx.conf 
```

#### 关闭nginx
```
nginx -s stop  快速停止nginx
         quit  完整有序的停止nginx
```

#### 启动nginx:
```
nginx -c /path/to/nginx.conf
```


#### 修改配置后重新加载生效
```
nginx -s reload 
```