# docker 快速搭建php开发环境

## 1. 安装nginx
```bash
docker pull nginx
```

## 2. 安装php
```
docker pull php:7.1.30-fpm
```

## 3. 实例化准备

新建几个文件夹，分别用来映射：网站根目录、`nginx` 配置文件、日志文件
```
mkdir -p ~/nginx/www ~/nginx/logs ~/nginx/conf
```

在新建的www目录中新建：`index.php` 用来检测php环境是否搭建成功:
```
<?php
    phpinfo();
```

**启动容器之前** 
在nginx配置文件目录 `conf` 下新建：`test-php.conf` ，后缀是`.conf`即可：
```
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm index.php;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    location ~ \.php$ {
        fastcgi_pass   php:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /www/$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```
配置说明
- php:9000: 表示 php-fpm 服务的 URL，下面我们会具体说明。
- /www/: 是 php-fpm 中 php 文件的存储路径，映射到本地的 ~/nginx/www 目录。

## 4. 启动 php
```bash
docker run --name  php7 -v ~/nginx/www:/www  -d php:7.1.30-fpm
```

## 5. 启动 nginx同时链接 php
```bash
docker run -it --name php-nginx -p 80:80 \
    -v ~/nginx/www:/usr/share/nginx/html \
    -v ~/nginx/conf:/etc/nginx/conf.d \
    --link php7:php -d nginx
```
- ~/nginx/www: 是本地 html 文件的存储目录，/usr/share/nginx/html 是容器内 html 文件的存储目录。
- ~/nginx/conf/conf.d: 是本地 nginx 配置文件的存储目录，/etc/nginx/conf.d 是容器内 nginx 配置文件的存储目录。
- --link myphp-fpm:php: 把 myphp-fpm 的网络并入 nginx，并通过修改 nginx 的 /etc/hosts，把域名 php 映射成 127.0.0.1，让 nginx 通过 php:9000 访问 php-fpm。

访问：`127.0.0.1`

## docker-compose 编排

创建 `docker-compose.yml` 文件
注意volume绑定的文件需要事先创建
```bash
services:
  web:
    image: nginx
    environment:
      - NGINX_HOST=foobar.com
      - NGINX_PORT=80
    ports:
      - "80:80"
    volumes:
      - ./nginx/www:/usr/share/nginx/html
      - ./nginx/conf/test.nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/conf:/etc/nginx/conf.d
      - ./nginx/logs:/var/log/nginx
    networks:
      - lnmp-network
      
  php:
    image: php:7.3-fpm
    volumes:
      - ./nginx/www:/www
    networks:
      - lnmp-network
  mysql:
    image: mysql
    ports:
      - "33061:33061"
    environment:
      - MYSQL_ROOT_PASSWORD=123456
    networks:
      - lnmp-network
      
networks:
  lnmp-network:
```

启动 `docker compose -d up`