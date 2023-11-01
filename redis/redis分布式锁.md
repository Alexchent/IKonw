# redis 分布式锁

分布式锁需要满足3个特性
1. 排他性，同一时刻只有一个客户端可以持有锁
2. 可重入性，避免死锁
3. 容错性

## 基础版
缺点: 不支持锁重入、不支持锁等待
### 关键命令
1. 加锁
value需要生成一个业务上的全局唯一ID
```
//key 锁定的资源
//value 请求id
set key value NX PX {毫秒} EX {秒}
```

解锁lua脚本 unlock.script
```
//先确定此时锁是否是自己持有的，在解锁
if redis.call("get", KEYS[1] == ARGV[1]) then
    return redis.call("del", KEYS[1])
else
    return 0
end
```

2. 执行解锁脚本,注意每个key、value之间需要用`,`隔开之间还有空格
```
redis-cli --eval unlock.script key , value
```

> [redis分布式锁 RedLock](https://github.com/ronnylt/redlock-php)

## redlock
1. 对多个redis节点请求加锁，超过半数的节点加锁成功，即认为加锁成功
2. 如果加锁失败，则对所有的节点解锁，注意用lua脚本解锁，核对value值，确保只解自己加上去的锁
3. 锁的有效期时，锁设置的有效期-真个加锁过程的耗时，在有效期内执行任务
4. 执行解锁

## redission 
[redission](./redis分布式锁redission.md)