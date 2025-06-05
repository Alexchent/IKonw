# 初认Kafka

## 1. kafka是什么
Apache Kafka 是一个开源**分布式事件流平台**

是最流行的开源**流处理**软件，用于**大规模**收集、处理、存储和分析数据。它以其卓越的性能、低延迟、容错和高吞吐量而闻名，它能够每秒处理数千条消息。

Kafka 起初是 由 LinkedIn 公司采用 Scala 语言开发的一个**多分区**、**多副本**且基于 **ZooKeeper** 协调的分布式消息系统，现已被捐献给 Apache 基金会。目前 Kafka 已经定位为一个**分布式** **流式处理**平台，它以高吞吐、可持久化、可水平扩展、支持流数据处理等多种特性而被广泛使用


Kafka 之所以受到越来越多的青睐，与它所“扮演”的三大角色是分不开的：

- **消息系统**： Kafka 和传统的消息系统（也称作消息中间件）都具备系统解耦、冗余存储、流量削峰、缓冲、异步通信等功能。与此同时，Kafka 还提供了大多数消息系统难以实现的<u>消息顺序性保障</u>及<u>回溯消费</u>的功能。
- **存储系统**： Kafka 把消息持久化到磁盘，相比于其他基于内存存储的系统而言，有效地降低了数据丢失的风险。也正是得益于 Kafka 的消息持久化功能和多副本机制，我们可以把 Kafka 作为长期的数据存储系统来使用，只需要把对应的数据保留策略设置为“永久”或启用主题的日志压缩功能即可。
- **流式处理平台**： Kafka 不仅为每个流行的流式处理框架提供了可靠的数据来源，还提供了一个完整的流式处理类库，比如窗口、连接、变换和聚合等各类操作。

## 2. 基本概念
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/3/5/16949bd6279df106~tplv-t2oaga2asx-watermark.awebp)

整个 Kafka 体系结构中引入了以下3个术语：

- **Producer**： 生产者，也就是发送消息的一方。生产者负责创建消息，然后将其投递到 Kafka 中。

- **Consumer**： 消费者，也就是接收消息的一方。消费者连接到 Kafka 上并接收消息，进而进行相应的业务逻辑处理。

- **Broker**： 服务代理节点。对于 Kafka 而言，Broker 可以简单地看作一个独立的 Kafka 服务节点或 Kafka 服务实例。大多数情况下也可以将 Broker 看作一台 Kafka 服务器，前提是这台服务器上只部署了一个 Kafka 实例。一个或多个 Broker 组成了一个 Kafka 集群。
- **event records** 事件记录了世界或您的业务中“发生了某事” 的事实。在文档中也称为记录或消息。当您向 Kafka 读取或写入数据时，您以事件的形式执行此操作。从概念上讲，事件具有键、值、时间戳和可选的元数据标头。这是一个示例事件：
    - event key: "Alice"
    - event value: "Made a payment of $200 to Bob"
    - event timestamp: "Jun. 25, 2020 at 2:06 p.m."
### topic 主体 和 Partition 分区

*先说结论：分区提升性能，副本保障稳定性*

在 Kafka 中还有两个特别重要的概念 — **主题**（Topic）与 **分区**（Partition）。Kafka 中的消息以主题为单位进行归类，生产者负责将消息发送到特定的主题（发送到 Kafka 集群中的每一条消息都要指定一个主题），而消费者负责订阅主题并进行消费。

主题是一个逻辑上的概念，它还可以细分为多个分区，一个分区只属于单个主题，很多时候也会把分区称为主题分区（Topic-Partition）。同一主题下的不同分区包含的消息是不同的，分区在存储层面可以看作一个可追加的日志（Log）文件，消息在被追加到分区日志文件的时候都会分配一个特定的偏移量（offset）。

**offset** 是消息在分区中的唯一标识，Kafka 通过它来保证消息在分区内的顺序性，不过 offset 并不跨越分区，也就是说，Kafka 保证的是分区有序而不是主题有序。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/3/5/16949cc96dc79e19~tplv-t2oaga2asx-watermark.awebp)

如上图所示，主题中有4个分区，消息被顺序追加到每个分区日志文件的尾部。Kafka 中的分区可以分布在不同的服务器（broker）上，也就是说，一个主题可以横跨多个 broker，以此来提供比单个 broker 更强大的性能。

**每一条消息被发送到 broker 之前，会根据 分区规则 选择存储到哪个具体的分区**。如果分区规则设定得合理，所有的消息都可以均匀地分配到不同的分区中。*如果一个主题只对应一个文件，那么这个文件所在的机器I/O将会成为这个主题的性能瓶颈，而分区解决了这个问题*。在创建主题的时候可以通过指定的参数来设置分区的个数，当然也可以在主题创建完成之后去修改分区的数量，通过增加分区的数量可以实现水平扩展。

### Replication 副本
Kafka 为分区引入了多**副本**（Replica）机制，通过增加副本数量可以提升容灾能力。

同一分区的不同副本中保存的是相同的消息（在同一时刻，副本之间并非完全一样），副本之间是“一主多从”的关系，其中 leader 副本负责处理读写请求，follower 副本只负责与 leader 副本的消息同步。副本处于不同的 broker 中，当 leader 副本出现故障时，从 follower 副本中重新选举新的 leader 副本对外提供服务。Kafka 通过多副本机制实现了故障的自动转移，当 Kafka 集群中某个 broker 失效时仍然能保证服务可用。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/3/5/16949c07c0df30dc~tplv-t2oaga2asx-watermark.awebp)

如上图所示，Kafka 集群中有4个 broker，某个主题中有3个分区，且副本因子（即副本个数）也为3，如此每个分区便有1个 leader 副本和2个 follower 副本。生产者和消费者只与 leader 副本进行交互，而 follower 副本只负责消息的同步，很多时候 follower 副本中的消息相对 leader 副本而言会有一定的滞后。

分区中的所有副本统称为 AR（Assigned Replicas）。所有与 leader 副本保持一定程度同步的副本（包括 leader 副本在内）组成ISR（In-Sync Replicas），ISR 集合是 AR 集合中的一个子集。消息会先发送到 leader 副本，然后 follower 副本才能从 leader 副本中拉取消息进行同步，同步期间内 follower 副本相对于 leader 副本而言会有一定程度的滞后。

前面所说的“一定程度的同步”是指可忍受的滞后范围，这个范围可以通过参数进行配置。与 leader 副本同步滞后过多的副本（不包括 leader 副本）组成 OSR（Out-of-Sync Replicas），由此可见，AR=ISR+OSR。在正常情况下，所有的 follower 副本都应该与 leader 副本保持一定程度的同步，即 AR=ISR，OSR 集合为空。


> [图解Kafka之实战指南](https://juejin.cn/book/6844733793220165639/section/6844733793572503559)
 
## 3. 优选副本的选举

分区使用多副本机制来提升可靠性，但只有 **leader** 副本对外提供读写服务，而 **follower** 副本只负责在内部进行消息的同步。如果一个分区的 leader 副本不可用，那么就意味着整个分区变得不可用，此时就需要 Kafka 从剩余的 follower 副本中挑选一个新的 leader 副本来继续对外提供服务。原 leader 副本的节点及时恢复也只会作为 follower 副本，就会造成集群负载不均衡。

在 Kafka 中可以提供分区自动平衡的功能，与此对应的 broker 端参数是 auto.leader. rebalance.enable，此参数的默认值为 true。

为了能够有效地治理负载失衡的情况，Kafka 引入了优先副本（preferred replica）的概念。所谓的优先副本是指在AR集合列表中的第一个副本。比如上面主题 topic-partitions 中分区0的AR集合列表（Replicas）为[1,2,0]，那么分区0的优先副本即为1。理想情况下，优先副本就是该分区的leader 副本，所以也可以称之为 preferred leader。Kafka 要确保所有主题的优先副本在 Kafka 集群中均匀分布，这样就保证了所有分区的 leader 均衡分布。如果 leader 分布过于集中，就会造成集群负载不均衡。

所谓的优先副本的选举是指通过一定的方式促使优先副本选举为 leader 副本，以此来促进集群的负载均衡，这一行为也可以称为“分区平衡”。

需要注意的是，**分区平衡并不意味着 Kafka 集群的负载均衡**，因为还要考虑集群中的分区分配是否均衡。更进一步，每个分区的 leader 副本的负载也是各不相同的，有些 leader 副本的负载很高，比如需要承载 TPS 为30000的负荷，而有些 leader 副本只需承载个位数的负荷。也就是说，就算集群中的分区分配均衡、leader 分配均衡，也并不能确保整个集群的负载就是均衡的，还需要其他一些硬性的指标来做进一步的衡量，这个会在后面的章节中涉及，本节只探讨优先副本的选举。

不过在生产环境中不建议将 `auto.leader.rebalance.enable` 设置为默认的 true，因为这可能引起负面的性能问题，也有可能引起客户端一定时间的阻塞

`kafka-leader-election.sh` 脚本提供了对分区 leader 副本进行重新平衡的功能。


## 4. 分区重新分配

需要进行分区迁移的场景：
- **集群扩容**
- **broker 节点失效**
 
`kafka-reassign-partitions.sh` 脚本的使用分为3个步骤：首先创建需要一个包含主题清单的 JSON 文件，其次根据主题清单和 broker 节点清单生成一份重分配方案，最后根据这份方案执行具体的重分配动作。

下面我们通过一个具体的案例来演示 kafka-reassign-partitions.sh 脚本的用法。首先在一个由3个节点（broker 0、broker 1、broker 2）组成的集群中创建一个主题 topic-reassign，主题中包含4个分区和2个副本：
```
[root@node1 kafka_2.11-2.0.0]# bin/kafka-topics.sh --zookeeper localhost:2181/ kafka --create --topic topic-reassign --replication-factor 2 --partitions 4
Created topic "topic-reassign".

[root@node1 kafka_2.11-2.0.0]# bin/kafka-topics.sh --zookeeper localhost:2181/ kafka --describe --topic topic-reassign
Topic:topic-reassign	PartitionCount:4	ReplicationFactor:2	Configs: 
    Topic: topic-reassign	Partition: 0	Leader: 0	Replicas: 0,2	Isr: 0,2
    Topic: topic-reassign	Partition: 1	Leader: 1	Replicas: 1,0	Isr: 1,0
    Topic: topic-reassign	Partition: 2	Leader: 2	Replicas: 2,1	Isr: 2,1
    Topic: topic-reassign	Partition: 3	Leader: 0	Replicas: 0,1	Isr: 0,1
```

我们可以观察到主题 topic-reassign 在3个节点中都有相应的分区副本分布。由于某种原因，我们想要下线 brokerId 为1的 broker 节点，在此之前，我们要做的就是将其上的分区副本迁移出去。使用 kafka-reassign-partitions.sh 脚本的第一步就是要创建一个 JSON 文件（文件的名称假定为 reassign.json），文件内容为要进行分区重分配的主题清单。对主题 topic-reassign 而言，示例如下：
```
{
        "topics":[
                {
                        "topic":"topic-reassign"
                }
        ],
        "version":1
}
```

第二步就是根据这个 JSON 文件和指定所要分配的 broker 节点列表来生成一份候选的重分配方案，具体内容参考如下：
```
[root@node1 kafka_2.11-2.0.0]# bin/kafka-reassign-partitions.sh --zookeeper localhost:2181/kafka --generate --topics-to-move-json-file reassign.json --broker-list 0,2
Current partition replica assignment
{"version":1,"partitions":[{"topic":"topic-reassign","partition":2,"replicas":[2,1],"log_dirs":["any","any"]},{"topic":"topic-reassign","partition":1,"replicas":[1,0],"log_dirs":["any","any"]},{"topic":"topic-reassign","partition":3,"replicas":[0,1],"log_dirs":["any","any"]},{"topic":"topic-reassign","partition":0,"replicas":[0,2],"log_dirs":["any","any"]}]}

Proposed partition reassignment configuration
{"version":1,"partitions":[{"topic":"topic-reassign","partition":2,"replicas":[2,0],"log_dirs":["any","any"]},{"topic":"topic-reassign","partition":1,"replicas":[0,2],"log_dirs":["any","any"]},{"topic":"topic-reassign","partition":3,"replicas":[0,2],"log_dirs":["any","any"]},{"topic":"topic-reassign","partition":0,"replicas":[2,0],"log_dirs":["any","any"]}]}
```

上面的示例中包含4个参数，其中 zookeeper 已经很常见了，用来指定 ZooKeeper 的地址。generate 是 kafka-reassign-partitions.sh 脚本中指令类型的参数，可以类比于 kafka-topics.sh 脚本中的 create、list 等，它用来生成一个重分配的候选方案。topic-to-move-json 用来指定分区重分配对应的主题清单文件的路径，该清单文件的具体的格式可以归纳为{"topics": [{"topic": "foo"},{"topic": "foo1"}],"version": 1}。broker-list 用来指定所要分配的 broker 节点列表，比如示例中的“0,2”。

上面示例中打印出了两个 JSON 格式的内容。第一个“Current partition replica assignment”所对应的 JSON 内容为当前的分区副本分配情况，在执行分区重分配的时候最好将这个内容保存起来，以备后续的回滚操作。第二个“Proposed partition reassignment configuration”所对应的 JSON 内容为重分配的候选方案，注意这里只是生成一份可行性的方案，并没有真正执行重分配的动作。生成的可行性方案的具体算法和创建主题时的一样，这里也包含了机架信息，具体的细节可以参考17节的内容。

我们需要将第二个 JSON 内容保存在一个 JSON 文件中，假定这个文件的名称为 project.json。

第三步执行具体的重分配动作，详细参考如下：
```
[root@node1 kafka_2.11-2.0.0]# bin/kafka-reassign-partitions.sh --zookeeper localhost:2181/kafka --execute --reassignment-json-file project.json 
Current partition replica assignment

{"version":1,"partitions":[{"topic":"topic-reassign","partition":2,"replicas":[2,1],"log_dirs":["any","any"]},{"topic":"topic-reassign","partition":1,"replicas":[1,0],"log_dirs":["any","any"]},{"topic":"topic-reassign","partition":3,"replicas":[0,1],"log_dirs":["any","any"]},{"topic":"topic-reassign","partition":0,"replicas":[0,2],"log_dirs":["any","any"]}]}

Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions.
```

我们再次查看主题 topic-reassign 的具体信息：
```
[root@node1 kafka_2.11-2.0.0]# bin/kafka-topics.sh --zookeeper localhost:2181/ kafka --describe --topic topic-reassign
Topic:topic-reassign	PartitionCount:4	ReplicationFactor:2	Configs: 
    Topic: topic-reassign	Partition: 0	Leader: 0	Replicas: 2,0	Isr: 0,2
    Topic: topic-reassign	Partition: 1	Leader: 0	Replicas: 0,2	Isr: 0,2
    Topic: topic-reassign	Partition: 2	Leader: 2	Replicas: 2,0	Isr: 2,0
    Topic: topic-reassign	Partition: 3	Leader: 0	Replicas: 0,2	Isr: 0,2
```

可以看到主题中的所有分区副本都只在0和2的 broker 节点上分布了。

在第三步的操作中又多了2个参数，execute 也是指令类型的参数，用来指定执行重分配的动作。reassignment-json-file 指定分区重分配方案的文件路径，对应于示例中的 project.json 文件。

分区重分配的**基本原理**是： 先通过控制器为每个分区添加新副本（增加副本因子），新的副本将从分区的 leader 副本那里复制所有的数据。根据分区的大小不同，复制过程可能需要花一些时间，因为数据是通过网络复制到新副本上的。在复制完成之后，控制器将旧副本从副本清单里移除（恢复为原先的副本因子数）。注意在重分配的过程中要确保有足够的空间。

对于分区重分配而言，这里还有可选的第四步操作，即验证查看分区重分配的进度。只需将上面的 execute 替换为 verify 即可，具体示例如下：
```
[root@node1 kafka_2.11-2.0.0]# bin/kafka-reassign-partitions.sh --zookeeper localhost:2181/kafka --verify --reassignment-json-file project.json 
Status of partition reassignment: 
Reassignment of partition topic-reassign-2 completed successfully
Reassignment of partition topic-reassign-1 completed successfully
Reassignment of partition topic-reassign-3 completed successfully
Reassignment of partition topic-reassign-0 completed successfully
```

分区重分配对集群的性能有很大的影响，需要占用额外的资源，比如网络和磁盘。在实际操作中，我们将**降低重分配的粒度**，分成多个小批次来执行，以此来将负面的影响降到最低，这一点和优先副本的选举有异曲同工之妙。 还需要注意的是，如果要将某个 broker 下线，那么在执行分区重分配动作之前**最好先关闭或重启 broker**。这样这个 broker 就不再是任何分区的 leader 节点了，它的分区就可以被分配给集群中的其他 broker。这样可以减少 broker 间的流量复制，以此提升重分配的性能，以及减少对集群的影响。

## ZooKeeper
ZooKeeper 是安装 Kafka 集群的必要组件，Kafka 通过 ZooKeeper 来实施对元数据信息的管理，包括集群、broker、主题、分区等内容。

ZooKeeper 是一个开源的分布式协调服务，是 Google Chubby的一个开源实现。分布式应用程序可以基于 ZooKeeper 实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、配置维护等功能。

在 ZooKeeper 中共有3个角色：leader、follower 和 observer，同一时刻 ZooKeeper 集群中只会有一个 leader，其他的都是 follower 和 observer。observer 不参与投票，默认情况下 ZooKeeper 中只有 leader 和 follower 两个角色。更多相关知识可以查阅 ZooKeeper 官方网站来获得。