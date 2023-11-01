# 过期key淘汰策略
## 1. 被动（惰性）删除
获取键时，检查是否过期（对CPU）友好；

## 2. 主动（定期）删除
Server cron事件，防止一些key被读取的概率极低，不会被动删除。定时执行Server cron事件（每秒10次，可以通过hz设置），随机检查清理设置了过期时间的键。对内存友好

对应配置
```
hz 10 //表示每秒运行10次,不要超过100，对CPU不友好
```

- 机测试100个设置了过期时间的key
- 删除所有发现的已过期的key
- 若删除的key超过25个则重复步骤1

## 3.当前已用内存超过maxmemory时，触发主动清理策略

- volatile-lru：只对设置了过期时间的key进行LRU（==默认值==）
- allkeys-lru ：删除lru算法的key
- volatile-lfu：只对设置了过期时间的key进行LFU（lru算法的增强版，key的最近使用时间和累计使用次数进行平衡）
- allkeys-lfu ：删除lfu算法的key
- volatile-random：随机删除即将过期key
- allkeys-random：随机删除
- volatile-ttl ：删除即将过期的
- noeviction ：永不过期，返回错误

> LRU Least Recently Used 最近使用的最少使用的

> LFU Least Frequently Used 使用频率最少的

对应配置
```
#最大内存，不设置表示无限制，通常设置为物理内存的75%
maxmemory <bytes>   

maxmemory-policy allkeys-lru
```

## 怎么选择淘汰策略

1. 如果redis仅作为缓存使用，即数据丢失也可以重建，那么选择allkeys-lfu，这个策略会有效的区分冷热数据
2. 如果数据要进行持久化，则建议使用volatile-lfu，当然最好是开辟专门的redis实例，一个用于缓存，一个用于持久化的数据

注意：避免触发第三种删除策略，可以提高主动删除的刷新时间，或进行扩容，因为redis的删除都是随机的