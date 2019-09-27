## 1、哨兵的介绍

sentinel，中文名是哨兵

哨兵是redis集群架构中非常重要的一个组件，主要功能如下

1. **集群监控**：负责监控redis master和slave进程是否正常工作
2. **消息通知**：如果某个redis实例有故障，那么哨兵负责发送消息作为报警通知给管理员
3. **故障转移：**如果master node挂掉了，会自动转移到slave node上
4. **配置中心：**如果故障转移发生了，通知client客户端新的master地址

哨兵用于实现 redis 集群的高可用，本身也是分布式的，作为一个哨兵集群去运行，互相协同工作

- 故障转移时，判断一个 master node 是否宕机，需要大部分哨兵同意才行，涉及到了<a href="#5、选举算法">分布式选举</a>的问题。
- 即使部分哨兵节点挂掉了，哨兵集群还是能正常工作的。

## 2、哨兵的核心知识

1. 哨兵至少需要 3 个实例，来保证自己的健壮性
2. 哨兵 + redis 主从的部署架构，是不会保证数据零丢失的，只能保证 redis 集群的高可用性
3. 对于哨兵 + redis 主从部署架构，尽量在测试环境和生产环境都进行充足的测试和演练

### 2.1、为什么至少需要3个实例？

首先，我们需要先了解 redis 哨兵的一些概念和重要参数

`sdown`：Subjective Down，主观宕机，就一个哨兵如果自己觉得一个master宕机了，那么就是主观宕机`

`odown`：Objective Down，客观宕机，如果`quorum`数量的哨兵都觉得一个master宕机了，那么就是客观宕机

>sdown达成的条件很简单，如果一个哨兵ping一个master，超过了 **is-master-down-after-milliseconds** 指定的毫秒数之后，就主观认为master宕机
>
>如果一个哨兵在指定时间内，收到了 quorum 数量的其它哨兵也认为那个 master 是 sdown 的，那么就认为是 odown 了。

`quorum`：只有当至少 `quorum` 个 sentinel 实例认为 master 宕机，才会认为 master 为 `odown`

`majority`：当某个 master 被认定为 odown 后，需要有 `majority`个哨兵来允许执行故障转移

> majority的取值： x / 2 + 1，比如 2的 majority 为2,3的 majority 为 2,5的majority为3
>
> 如果 quorum < majority，比如 5 个哨兵，majority 就是 3，quorum 设置为 2，那么就 3 个哨兵授权就可以执行切换。
>
> 但是如果 quorum >= majority，那么必须 quorum 数量的哨兵都授权，比如 5 个哨兵，quorum 是 5，那么必须 5 个哨兵都同意授权，才能执行切换。

学习了上面的一些概念与参数，接下来的例子我们就很好理解了

如果哨兵集群仅仅部署了两个哨兵实例，`quorum = 1`

```shell
+----+         +----+
| M1 |---------| R1 |
| S1 |         | S2 |
+----+         +----+
```

若 master 宕机，s1 和 s2 中只要有 1 个哨兵认为 master 宕机了，就可以进行切换，同时 s1 和 s2 会选举出一个哨兵来执行故障转移。但是这里需要有 `majority`个哨兵是运行的。

前面我们已经知道了，2个 redis哨兵实例的 majority 为 2

如果此时仅仅是 M1 进程宕机了，哨兵 s1 正常运行，那么故障转移是 ok的。但是如果是整个 M1 和 S1 运行的机器宕机了，那么哨兵只有 1 个，此时就没有 majority 来允许执行故障转移，虽然另外一台机器上还有一个 R1，但是故障转移不会执行。

经典的 3 节点哨兵集群是这样的：

```shell
  	   +----+
       | M1 |
       | S1 |
       +----+
          |
+----+    |    +----+
| R2 |----+----| R3 |
| S2 |         | S3 |
+----+         +----+
```

配置 `quorum = 2`，如果 M1 所在机器宕机了，那么三个哨兵还剩下两个，S2 和 S3 可以一致认为 master 宕机了，然后选举出一个来执行故障，同时 3 个哨兵的 majority 是 2，所以还剩下的2个哨兵运行着，就可以允许执行故障转移。

## 3、哨兵架构下丢失数据的情况

在进行主备切换的过程中，可能会导致数据丢失。

### 3.1、异步复制导致的数据丢失

因为 master -> slave 的复制是异步的，所以可能有部分数据还没有复制到 slave，master 就宕机了，此时这部分数据就丢失了

### 3.2、集群脑裂导致的数据丢失

脑裂，也就是说，某个 master 所在的机器突然脱离了正常的网络，跟其他 slave 无法连接，但是实际上该 master 还运行着， 此时哨兵可能就会认为 master 宕机了，然后开启选举，将其他 salve 切换成 master

这个时候，集群里就有两个 master了，也就是所谓的脑裂。

此时虽然某个 slave 被切换成了 master，但是可能 client 还没来得及切换到新的 master，还继续向旧的 master 写入数据，那可能这部分数据就丢失了。因为当网络恢复后，旧 master 重新连接后会被作为一个 slave 挂到新的master 上，自己的数据会被清空，然后重新从新的 master 上复制数据。

### 3.3、解决方案

对 redis.conf 进行如下配置

```shell
min-slaves-to-write 1 #至少有1个slave
min-slaves-max-lag 10 #数据复制和同步延迟不能超过10s
```

如果说一旦所有的 slave，数据复制和同步的延迟都超过了 10s，那么这个时候 ，master 就不会再接受任何请求了

- 减少对异步复制数据的丢失

有了 `min-slaves-max-lag` 这个配置，就可以确保说，一旦 slave 复制数据和 ack 延时太长，就认为可能 master 宕机后损失的数据太多了，那么就拒绝写请求，这样可以把 master 宕机时由于部分数据未同步到 slave 导致的数据丢失降低的可控范围内。

- 减少脑裂的数据丢失

如果一个 master 出现了脑裂，跟其他 slave 丢了连接，那么上面两个配置可以确保说，如果不能继续给指定数量的 slave 发送数据，而且 slave 超过 10 秒没有给自己 ack 消息，那么就直接拒绝客户端的写请求。因此在脑裂场景下，最多就丢失 10 秒的数据。

## 4、哨兵集群的自动发现机制

哨兵互相之间的发现，是通过 redis 的 `pub/sub` 系统实现的，每个哨兵都会往 `__sentinel__:hello` 这个 channel 里发送一个消息，这时候所有其他哨兵都可以消费到这个消息，并感知到其他的哨兵的存在。

每隔两秒钟，每个哨兵都会往自己监控的某个 master+slaves 对应的 `__sentinel__:hello` channel 里**发送一个消息**，内容是自己的 host、ip 和 runid 还有对这个 master 的监控配置。

每个哨兵也会去**监听**自己监控的每个 master+slaves 对应的 `__sentinel__:hello` channel，然后去感知到同样在监听这个 master+slaves 的其他哨兵的存在。

每个哨兵还会跟其他哨兵交换对 `master` 的监控配置，互相进行监控配置的同步。

**slave 配置的自动纠正**：哨兵会负责自动纠正 slave 的一些配置，比如 slave 如果要成为潜在的 master 候选人，哨兵会确保 slave 复制现有 master 的数据；如果 slave 连接到了一个错误的 master 上，比如故障转移之后，那么哨兵会确保它们连接到正确的 master 上。

## 5、选举算法

如果一个 master 被认为 odown 了，而且 majority 数量的哨兵都允许主备切换，那么某个哨兵就会执行主备切换操作，此时首先要选举一个 slave 来，会考虑 slave 的一些信息：

- 跟 master 断开连接的时长
- slave 优先级
- 复制 offset
- run id

如果一个 slave 跟 master 断开连接的时间已经超过了 `down-after-milliseconds` 的 10 倍，外加 master 宕机的时长，那么 slave 就被认为不适合选举为 master。

```shell
(down-after-milliseconds * 10) + milliseconds_since_master_is_in_SDOWN_state
```

接下来会对 slave 进行排序：

- 按照 slave 优先级进行排序，slave priority 越低，优先级就越高。
- 如果 slave priority 相同，那么看 replica offset，哪个 slave 复制了越多的数据，offset 越靠后，优先级就越高。
- 如果上面两个条件都相同，那么选择一个 run id 比较小的那个 slave。

### configuration epoch

哨兵会对一套 redis master+slaves 进行监控，有相应的监控的配置。

执行切换的那个哨兵，会从要切换到的新 master（salve->master）那里得到一个 configuration epoch，这就是一个 version 号，每次切换的 version 号都必须是唯一的。

如果第一个选举出的哨兵切换失败了，那么其他哨兵，会等待 failover-timeout 时间，然后接替继续执行切换，此时会重新获取一个新的 configuration epoch，作为新的 version 号。

### configuration 传播

哨兵完成切换之后，会在自己本地更新生成最新的 master 配置，然后同步给其他的哨兵，就是通过之前说的 `pub/sub` 消息机制。

这里之前的 version 号就很重要了，因为各种消息都是通过一个 channel 去发布和监听的，所以一个哨兵完成一次新的切换之后，新的 master 配置是跟着新的 version 号的。其他的哨兵都是根据版本号的大小来更新自己的 master 配置的。

## 6、哨兵集群的搭建

### 6.1、安装 Redis

### 6.2、配置

哨兵默认用26379端口，默认不能跟其他机器在指定端口连通，只能在本地访问

```shell
mkdir /etc/sentinel/   #存放 sentinel.conf配置文件
mkdir -p /var/sentinel/5000 

cp /usr/local/redis-3.2.8/sentinel.conf /etc/sentinel/5000.conf
vi /etc/sentinel/5000.conf

daemonize yes
logfile /var/log/sentinel/5000/sentinel.log
port 5000
bind 10.116.9.8   #本机IP地址
dir /var/sentinal/5000 #working-directory
#sentinel monitor <master-name> <ip> <redis-port> <quorum>
#master-name 指定监控的master名称
#ip 监控的master ip地址
#redis-port 监控的 redis-port
#quorum 配置 quorum 参数
sentinel monitor mymaster 10.116.9.8 6379 2
#超过30s与一个redis实例断开连接，哨兵就可能认为这个redis实例挂了
sentinel down-after-milliseconds mymaster 30000
#执行故障转移的timeout时长
sentinel failover-timeout mymaster 60000
#新的master被切换之后，同时有多少个slave被切换到去连接新master，重新做同步，数字越低，花费的时间越多
sentinel parallel-syncs mymaster 1
```

以上是一台机器上的 sentinel.conf的配置，其他两台机器的配置只需要修改对应的ip地址即可

```shell
daemonize yes
logfile /var/log/sentinel/5000/sentinel.log

port 5000
bind 10.116.9.12
dir /var/sentinal/5000
sentinel monitor mymaster 10.116.9.8 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1 
```

```shell
daemonize yes
logfile /var/log/sentinel/5000/sentinel.log

port 5000
bind 10.116.9.49
dir /var/sentinal/5000
sentinel monitor mymaster 10.116.9.8 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1 
```

### 6.3、启动哨兵进程

在eshop-cache01、eshop-cache02、eshop-cache03三台机器上，分别启动三个哨兵进程，组成一个集群，观察一下日志的输出

```shell
redis-sentinel /etc/sentinel/5000.conf
#redis-server /etc/sentinal/5000.conf --sentinel
```

日志里会显示出来，每个哨兵都能去监控到对应的redis master，并能够自动发现对应的slave

哨兵之间，互相会自动进行发现，用的就是之前说的pub/sub，消息发布和订阅channel消息系统和机制

### 6.4、检查哨兵状态

```shell
redis-cli -h 10.116.9.1 -p 5000

sentinel master mymaster
SENTINEL slaves mymaster
SENTINEL sentinels mymaster

SENTINEL get-master-addr-by-name mymaster
```

## 7、容灾演练

通过哨兵看一下当前的master

`SENTINEL get-master-addr-by-name mymaster`

把master节点kill -9掉，不要删了哨兵了，pid文件也删除掉(/var/run)

```shell
rm -rf /var/run/redis.pid
```

查看sentinal的日志，是否出现+sdown字样，识别出了master的宕机问题; 然后出现+odown字样，就是指定的quorum哨兵数量，都认为master宕机了

>1. 三个哨兵进程都认为 master 是 sdown 了
>2. 超过 quorum 指定的哨兵进程都认为 sdown 之后，就变为 odown
>3. 哨兵1是被选举为要执行后续的主备切换的那个哨兵
>4. 哨兵1去新的master（slave）获取了一个新的config version
>5. 尝试执行 failover
>6. 投票选举出一个 slave 区切换成 master ，每隔哨兵都会执行一次投票
>7. 让salve，slaveof noone，不让它去做任何节点的slave了; 把slave提拔成master; 旧的master认为不再是master了
>8. 哨兵就自动认为之前的9:6379变成了slave了，12:6379变成了master了
>9. 哨兵去探查了一下12:6379这个salve的状态，认为它sdown了

所有哨兵选举出了一个，来执行主备切换操作

如果哨兵的majority都存活着，那么就会执行主备切换操作

再通过哨兵看一下master：SENTINEL get-master-addr-by-name mymaster

尝试连接一下新的master

故障恢复，再将旧的master重新启动，查看是否被哨兵自动切换成slave节点

（1）手动杀掉master
（2）哨兵能否执行主备切换，将slave切换为master
（3）哨兵完成主备切换后，新的master能否使用
（4）故障恢复，将旧的master重新启动
（5）哨兵能否自动将旧的master变为slave，挂接到新的master上面去，而且也是可以使用的