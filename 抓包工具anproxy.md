# 抓包工具anyroxy使用

## 安装
1. 安装node，[node官网](https://nodejs.org/en)下载

新版node自带npm，检查npm是否安装好
```
npm -v
```

2. 安装anyproxy

```
sudo npm install anyproxy -g
```

## 启动anyproxy

```bash
-i 表示代理https
-p 代理服务的端口号
-w 可视化页面
anyproxy -i -p 8001 -w 8002
```

开启https，需要生成证书
```
anyproxy-ca
```

## 自定义规则文件
编写规则文件 rule.js
```js
module.exports = {
    shouldInterceptHttpsReq : function(req){
        return true;
    }
}
```

启动并加载规则
```
anproxy -i -r rule.js
```

测试规则
```
# 直接请求服务器
curl https://github.com

# 通过代理服务器请求
curl https://github.com --proxy http://127.0.0.1:8001
```

## 卸载

1. 清除证书
```
anyproxy --clear
```

2. 卸载
```
npm uninstall anyproxy
```

访问 `127.0.0.1:8002` 手机上扫二维码下载证书
![](assets/17003929947834.jpg)


参考文章
> https://www.jianshu.com/p/2074f7572694

# pm2管理anyproy

> [PM2 命令使用方法总结](https://juejin.cn/post/6889300755539312653)
>
> [electron-anyproxy](https://github.com/fwon/electron-anyproxy)
> 一个网络代理客户端，依赖于anyproxy是，构建在election和vue之上