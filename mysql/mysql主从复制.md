# mysql 主从复制

## 主从复制步骤
1. 主库将所有的写操作记录在**binlog**日志中，并生成**log dump线程**将binlog日志传给从库的I/O线程
2. 从库生成两个线程，一个是**I/O线程**，另一个是**SQL线程**
2. I/O线程去请求主库的binlog日志，并将binlog日志中的文件写入relay log（中继日志）中
3. SQL线程会读取relay loy中的内容，并解析成具体的操作，来实现主从的操作一致，达到最终数据一致的目的

## 主从复制配置步骤：
确保从数据库与主数据库里的数据一致，同步数据（mysqldump导出，source导入）
在主数据库里创建一个同步账户授权给从数据库使用
配置主数据库（修改配置文件）
配置从数据库（修改配置文件）或用sql命令设置

Master
1. 备份主库，注意**另开一个客户端给数据库上读锁**，避免在备份期间有其他人在写入导致数据同步不一致。
2. 首先master服务器配置开启binary log二进制日志，并重启
3. master上创建一个专门用来做主从同步到账户

Slave
1. 使用scp或rsync等工具，把master备份出来的数据传到slave服务器，并执行source导入数据
2. slave服务器配置启动一个I/O线程，通过配置好的账号连接到主服务器请求读取binlog，然后把读取到的二进制文件写道本地的Relay log（中继日志）中
4. slave服务器同时开启一个SQL线程，定时检查Relay log（也是二进制文件），如果发现更新立即把更新的内容在本地数据库上执行一遍。

每个slave都会从主服务器收到二进制日志的全部内容副本。slave设备负责决定应该执行二进制日志中的哪些语句。


## 具体操作：
master
1. 备份master上的数据，注意另开一个会话加读锁，备份完成后才能关闭这个会话终端
```
mysql> flush tables with read lock;
Query OK, 0 rows affected (0.76 sec)
```
此锁表的终端必须在备份完成以后才能退出（退出锁表失效）

```
mysqldump  -u用户名  -p密码  --all-databases  --master-data=1 > dbdump.db
```

如果不使用 --master-data 参数，则需要手动锁定单独会话中的所有表

2. 修改master的数据库配置文件**my.cnf**,开启binary log

`log-bin=/home/log/mysql-bin`可以自由选择binlog文件的位置，但要确保有权限
`service-id=1`

注意：
如果省略server-id（或将其显式设置为默认值0），则主服务器拒绝来自从服务器的任何连接。
为了在使用带事务的InnoDB进行复制设置时尽可能提高持久性和一致性，您应该在master my.cnf文件中使用以下配置项：
```
innodb_flush_log_at_trx_commit = 1
sync_binlog = 1
```

确保在主服务器上 skip_networking 选项处于 OFF 关闭状态, 这是默认值。
如果是启用的，则从站无法与主站通信，并且复制失败。
```
mysql> show variables like '%skip_networking%';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| skip_networking | OFF   |
+-----------------+-------+
1 row in set (0.00 sec)
```

1. 创建一个给与replication slave权限的账户
```
mysql> create user 'repl'@'192.168.55.129' identified by '123456';
Query OK, 0 rows affected (5.50 sec)

mysql> grant replication slave on *.* to 'repl'@'192.168.55.129';
Query OK, 0 rows affected (0.04 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.09 sec)
```
4. 修改slave的数据库配置文件my.cnf
server-id=2; #多从server—id要唯一且比master小
slave
1. 导入master的备份数据
2. 在slave数据库执行sql命令或直接在my.cnf中配置
```
CHANGE MASTER TO 
MASTER_HOST='125.564.12.1',   #master服务器ip
MASTER_PORT=3306,		#master mysql服务端口号
MASTER_USER='joe',                    #之前在master上创建的账户名
MASTER_PASSWORD='secret';

start slave;      				#启动从服务器的复制进程
```

将新的服务器加入，变为从服务器。

和上面的步骤一样，但是新加入的服务器的server-id 的值不能和现有都服务器 server-id 的值一样。


# 主从不一致
用户注册后，登录不了

### 解决方案
1. 开启半同步复制 semisync，即数据至少同步到一个从库才返回响应。该方案最简单，无需修改业务代码，但是数据库写性能降低
2. 强制读主，预估业务中写死，无法预防突发情况
3. 数据库连接路由，用缓存记录500ms内发生写操作的table，读操作通过该缓存判断是否路由到主库