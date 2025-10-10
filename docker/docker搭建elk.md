# docker 搭建elk 

“ELK”是三个开源项目的首字母缩写，这三个项目分别是：Elasticsearch、Logstash 和 Kibana。Elasticsearch 是一个搜索和分析引擎。Logstash 是服务器端数据处理管道，能够同时从多个来源采集数据，转换数据，然后将数据发送到诸如 Elasticsearch 等“存储库”中。Kibana 则可以让用户在 Elasticsearch 中使用图形和图表对数据进行可视化。

Elastic Stack 是 ELK Stack 的更新换代产品。

## 1. 利用 [sebp/elk](https://hub.docker.com/r/sebp/elk) 搭建
> 官方文档 https://elk-docker.readthedocs.io/#usage

### 注意事项
1. 至少要分配4G内存给docker

### 开始
1. 下载镜像 `docker pull sebp/elk:7.13.2`
2. 第一次启动容器 `docker run -it -d -p 5601:5601 -p 9200:9200 -p 5044:5044 --name elk sebp/elk:7.13.2`
3. 停止容器 `docker stop {container-id}`
4. 再次启动 `docker start elk`
5. 访问`127.0.0.1:5601` 就可以看到kinana界面啦

### 用docker-compose.yml启动

1. 创建`docker-compose.yml`文件
```
cd ~/docker/elk
vim docker-compose.yml
```
文件的内容如下：
```
elk:
  image: sebp/elk:7.13.2
  ports:
    - "5601:5601"
    - "9200:9200"
    - "5044:5044"
```
2. 该文件的同级目录下，启动 `docker-compose up elk`

## 2. 创建虚拟日志，来看看效果吧

1. 先看看我们启动的容器 `docker ps` 找到容器id
2. 进入该容器 `docker exec -it {container_id} /bin/bash`， 注意 {container_id} 替换成你的容器id
3. 进入容器后执行下面的命令
```
/opt/logstash/bin/logstash --path.data /tmp/logstash/data \
    -e 'input { stdin { } } output { elasticsearch { hosts => ["localhost"] } }'
```
4. 等待 Logstash 启动（如消息所示The stdin plugin is now waiting for input:），然后键入一些虚拟文本，然后按 Enter 以创建日志条目：
```
this is a dummy entry
```
![](assets/16304032207093.jpg)

5. 如果您浏览到`http://<your-host>:9200/_search?pretty&size=1000`（例如http://localhost:9200/_search?pretty&size=1000 用于本地本地本地 Docker 实例），您将看到 Elasticsearch 已将条目编入索引：
![](assets/16304033427189.jpg)

6. 接下来去kibnana看看日志

导航：Home -> Manage -> Index patterns -> Create index pattern -> ...
![](assets/16304034672575.jpg)

![](assets/16304035949788.jpg)
![](assets/16304036942967.jpg)如图：我们可以看到目前已经有2个数据源，我们根据自己的需要创建一个`logstash*`可以同事匹配这两个数据源
![](assets/16304038799658.jpg)
这样index pattern就创建好了，接下来我倒discover看看前面写入的日志吧。根据message筛选结果如下：

![](assets/16304039889729.jpg)

以上表名elk搭建完成并正常运行了

## 3. 用filebeat采集nginx日志到elk中

### 官方给了一个example，我们来试试
```
git clone https://github.com/spujadas/elk-docker.git

cd elk-docker/nginx-filebeat
```

关键文件 `filebeat.yml`
```
output:
  logstash:
    enabled: true
    hosts:
      - elk:5044
    timeout: 15
    tls:
      certificate_authorities:
      - /etc/pki/tls/certs/logstash-beats.crt

filebeat:
  prospectors:
    -
      paths:
        - /var/log/syslog
        - /var/log/auth.log
      document_type: syslog
    -
      paths:
        - "/var/log/nginx/*.log"
      document_type: nginx-access
```

### 修改start.sh，同样，把elk替换成ELK服务的hostname或ip
```
#!/bin/bash

curl -XPUT 'http://elk:9200/_template/filebeat?pretty' -d@/etc/filebeat/filebeat.template.json
/etc/init.d/filebeat start
nginx
tail -f /var/log/nginx/access.log -f /var/log/nginx/error.log
```

### 构建images
```
docker build -t alex/filebeat-nginx-example .
```

### 启动容器
注意link我们之前启动elk服务容器
```
docker run -p 80:80 -it --link elk \
    --name filebeat-nginx-example alex/filebeat-nginx-example
```

注意前面的filebeat.yml，将elk:5044中的elk替换成ELK服务端的hostname或ip。
### 接下来去kibana看看吧