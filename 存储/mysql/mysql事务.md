# MYSQL事务

## 隐式事务&显式事务
显示事务，用于一组sql操作。关键字：
```
begin
...
rollback
...
commit 
```

隐式事务，用于单个sql操作

> 事务是一个预SQL工具。通常事务包含一系列SQL操作，保证这些操作，全部成功则提交`commit`；如果有一项失败则所有操作回滚`rollback`。

注意点：只有使用了`InnoDB`引擎的库或表才支持事务

### 事务的四种特性ACID：
- **原子性**（atomicity）：一个事务（transaction），要么全部完成（commit），要么全部回滚（rollback）。
- **一致性**（consistenty）：事务开始前和事务结束后，数据库的完整性没有被破坏
- **隔离性**（isolation）：数据库允许多个并发事务同时对数据库进行读写和修改。隔离性可以防止多个事务并发执行时由于交叉执行而导致数据不一致。
- **持久性**：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

### 事务4中隔离级别

1. 读未提交（Read uncommited）：一个事务可以读取另一个事务未提交的数据。导致**脏读**
2. 读提交（Read commited）：一个事务要等待另一个事物提交才能读数据，解决脏读可能出现**不可重复读**即两次相同的查询结果不一致。大多数数据库默认这个隔离级别
3. 可重复读（repeatable read）：开始读取数据时，其他事物不允许修改。解决不可重复读，很大程度避免幻读，但仍有**可能**出现**幻读**。mysql默认这个级别
4. 串行化（serializable）：最高隔离级别。事物串行化执行，可避免脏读、不可重复读、幻读。但是效率低下，比较消耗数据库性能，一般不使用。

#### 脏读：
读另一个事务未提交的修改，后这个事务回滚了或做了其他修改，那这次读到的结果就是脏读

#### 不可重复读：
“不可重复读”现象发生在当执行SELECT 操作时没有获得读锁或者SELECT操作执行完后马上释放了读锁； 另外一个事务对数据进行了更新,读到了不同的结果.

![](https://picx.zhimg.com/v2-3c6b49e0d01717d6e4f2347329ba7a41_720w.jpg?source=d16d100b)
在这个例子中，事务2提交成功，因此他对id为1的行的修改就对其他事务可见了。导致了事务1在此前读的age=1,第二次读的age=2,两次结果不一致,这就是不可重复读.

#### 幻读：
针对**范围查**出现的情况。举例：查询价格大于1000的总金额。select sum(price) from order where price > 1000;
因为行锁针对的是已存在的数据行。并不会限制符合该条件的数据行增加，如果有新的金额大于1000的订单出现。再执行上面的sql结果就不一样了，这种情况我们就是幻读。


### 幻读怎么解决
> MySQL InnoDB 引擎的默认隔离级别虽然是「可重复读」，但是它很大程度上避免幻读现象（并不是完全解决了），解决的方案有两种：

- **针对快照读（普通 select 语句），是通过 MVCC 方式解决了幻读**，因为可重复读隔离级别下，事务执行过程中看到的数据，一直跟这个事务启动时看到的数据是一致的，即使中途有其他事务插入了一条数据，是查询不出来这条数据的，所以就很好了避免幻读问题。

- **针对当前读（select ... for update 等语句），是通过 next-key lock（记录锁+间隙锁）方式解决了幻读**，因为当执行 select ... for update 语句的时候，会加上 next-key lock，如果有其他事务在 next-key lock 锁范围内插入了一条记录，那么这个插入语句就会被阻塞，无法成功插入，所以就很好了避免幻读问题

### 什么样的情况会出现幻读
1. **RR读不会受到其他事务update、insert的影响，但是自己执行了update就会把其他事务insert的数据更新成自己的版本号，下一次读取就会读到了**。因此需要手动启动当前读，select语句后面加上`for update` 或 `lock in share mode`,这样等同于将隔离基本提升到了串行化
2. A事务开始后，先快照读，后当前读，如果期间其他事务有增删操作，就会导致当前读的结果与上一次的快照读不一致


一个user表
```sql
CREATE TABLE `user` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `class_id` int NOT NULL,
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_class_id` (`class_id`)
) ENGINE=InnoDB 
```

| 时序|  事务A | 事务B |
|---|---|---|
| 1 | select * from user where class_id=1; |  |
| 2 |  | insert into user values(1,1,"tony"); commit; | |
| 3|  select * from user where class_id=1;|  |
| 4| update user set name="hello" |  |
| 5| select * from user where class_id=1; |  |




### 当前读、快照读
> 当前读, 读取的是最新版本, 并且对读取的记录加锁, 阻塞其他事务同时改动相同记录，避免出现安全问题。

例如，假设要update一条记录，但是另一个事务已经delete这条数据并且commit了，如果不加锁就会产生冲突。所以update的时候肯定要是当前读，得到最新的信息并且锁定相应的记录。

> 快照读：
简单的select操作(不包括 select … lock in share mode, select … for update)。
Read Committed隔离级别：每次select都生成一个快照读。
Read Repeatable隔离级别：开启事务后第一个select语句才是快照读的地方，而不是一开启事务就快照读。

在RR级别下，快照读是通过**MVVC**(多版本控制)和**undo log**来实现的，当前读是通过加**record lock**(记录锁)和**gap lock**(间隙锁)来实现的


### MVCC 多版本并发控制
在InnoDB中，会在每行数据后添加两个额外的隐藏的值来实现MVCC，这两个值一个记录这行数据何时被创建，另外一个记录这行数据何时过期（或者被删除）。在实际操作中，存储的并不是时间，而是事务的版本号，每开启一个新事务，事务的版本号就会递增。 在可重读Repeatable reads事务隔离级别下：

- SELECT时，读取创建版本号<=当前事务版本号，删除版本号为空或>当前事务版本号。
- INSERT时，保存当前事务版本号为行的创建版本号
- DELETE时，保存当前事务版本号为行的删除版本号
- UPDATE时，插入一条新纪录，保存当前事务版本号为行创建版本号，同时保存当前事务版本号到原来删除的行
