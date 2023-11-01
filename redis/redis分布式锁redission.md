# redis分布式锁——redission(完美解决方案)
特性：
1. 互斥性
2. 防死锁
3. 加锁解锁必须为同一客户端
4. 容错性，大多数redis节点正常运行，客户端就可以加解锁

特点
1. 利用lua脚本保证加锁和解锁的原子性
2. 利用 pub/sub 机制，让竞争者等待一个可容忍的时间获取锁，减少重复请求

加锁lua脚本

|参数|	示例值|含义|
|---|---|---|
|KEY个数|	1|	KEY个数|
|KEYS[1]|	my_first_lock_name	|锁名
|ARGV[1]	| 60000	|持有锁的有效时间：毫秒
|ARGV[2]	| 58c62432-bb74-4d14-8a00-9908cc8b828f:1|	唯一标识：获取锁时set的唯一值，实现上为redisson客户端ID(UUID)+线程ID
```lua
-- 若锁不存在：则新增锁，并设置锁重入计数为1、设置锁过期时间
if (redis.call('exists', KEYS[1]) == 0) then
    redis.call('hset', KEYS[1], ARGV[2], 1);
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return nil;
end;
 
-- 若锁存在，且唯一标识也匹配：则表明当前加锁请求为锁重入请求，故锁重入计数+1，并再次设置锁过期时间
if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then
    redis.call('hincrby', KEYS[1], ARGV[2], 1);
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return nil;
end;
 
-- 若锁存在，但唯一标识不匹配：表明锁是被其他线程占用，当前线程无权解他人的锁，直接返回锁剩余过期时间
return redis.call('pttl', KEYS[1]);
```

解锁

|参数|	示例值	|含义
|---|---|---|
|KEY个数	|2|	KEY个数
KEYS[1]	|my_first_lock_name	|锁名
KEYS[2]	|redisson_lock__channel:{my_first_lock_name}|	解锁消息PubSub频道
ARGV[1]	|0	|redisson定义0表示解锁消息
ARGV[2]	|30000	|设置锁的过期时间；默认值30秒
ARGV[3]	|58c62432-bb74-4d14-8a00-9908cc8b828f:1|	唯一标识；同加锁流程

脚本内容
```lua
-- 若锁不存在：则直接广播解锁消息，并返回1
if (redis.call('exists', KEYS[1]) == 0) then
    redis.call('publish', KEYS[2], ARGV[1]);
    return 1; 
end;
 
-- 若锁存在，但唯一标识不匹配：则表明锁被其他线程占用，当前线程不允许解锁其他线程持有的锁
if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then
    return nil;
end; 
 
-- 若锁存在，且唯一标识匹配：则先将锁重入计数减1
local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); 
if (counter > 0) then 
    -- 锁重入计数减1后还大于0：表明当前线程持有的锁还有重入，不能进行锁删除操作，但可以友好地帮忙设置下过期时期
    redis.call('pexpire', KEYS[1], ARGV[2]); 
    return 0; 
else 
    -- 锁重入计数已为0：间接表明锁已释放了。直接删除掉锁，并广播解锁消息，去唤醒那些争抢过锁但还处于阻塞中的线程
    redis.call('del', KEYS[1]); 
    redis.call('publish', KEYS[2], ARGV[1]); 
    return 1;
end;
 
return nil;
```

步骤：
1. 第一步通过lua脚本获取锁
2. 第二步如果没有获取到锁，则订阅解锁消息，调用信号量方法阻塞住，直到被唤醒或等待超时
3. 一旦释放锁，就会广播消息，监听器就会释放信号，被阻塞的线程就会重新尝试获取锁
 
