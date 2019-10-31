## 1、redis cluster 介绍

- 自动将数据进行分片，每个 master 上放一部分数据
- 提供内置的高可用支持，部分 master 不可用时，还是可以继续工作的

在 redis cluster 架构下，每个 redis 要放开两个端口号，比如一个是 6379，另外一个就是 加1w 的端口号，比如 16379。

16379 端口号是用来进行节点间通信的，也就是 cluster bus 的东西，cluster bus 的通信，用来进行故障检测、配置更新、故障转移授权。cluster bus 用了另外一种二进制的协议，`gossip` 协议，用于节点间进行高效的数据交换，占用更少的网络带宽和处理时间。

### 节点间的内部通信机制

#### 基本通信原理

集群元数据的维护有两种方式：集中式、Gossip 协议。redis cluster 节点间采用 gossip 协议进行通信。

**集中式**是将集群元数据（节点信息、故障等等）几种存储在某个节点上。集中式元数据集中存储的一个典型代表，就是大数据领域的 `storm`。它是分布式的大数据实时计算引擎，是集中式的元数据存储的结构，底层基于 zookeeper（分布式协调的中间件）对所有元数据进行存储维护。

![](D:\File\studyNote\images\1568857221(1).jpg)

redis 维护集群元数据采用另一个方式， `gossip` 协议，所有节点都持有一份元数据，不同的节点如果出现了元数据的变更，就不断将元数据发送给其它的节点，让其它节点也进行元数据的变更。

![](../images/redis-gossip.png)

**集中式**的**好处**在于，元数据的读取和更新，时效性非常好，一旦元数据出现了变更，就立即更新到集中式的存储中，其它节点读取的时候就可以感知到；**不好**在于，所有的元数据的更新压力全部集中在一个地方，可能会导致元数据的存储有压力。

gossip 好处在于，元数据的更新比较分散，不是集中在一个地方，更新请求会陆陆续续打到所有节点上去更新，降低了压力；不好在于，元数据的更新有延时，可能导致集群中的一些操作会有一些滞后。

- 10000 端口：每个节点都有一个专门用于节点间通信的端口，就是自己提供服务的端口号+10000，比如 7001，那么用于节点间通信的就是 17001 端口。每个节点每隔一段时间都会往另外几个节点发送 `ping` 消息，同时其它几个节点接收到 `ping` 之后返回 `pong`。
- 交换的信息：信息包括故障信息，节点的增加和删除，hash slot 信息等等。

#### gossip 协议

gossip 协议包含多种消息，包含 `ping`,`pong`,`meet`,`fail` 等等。

- meet：某个节点发送 meet 给新加入的节点，让新节点加入集群中，然后新节点就会开始与其它节点进行通信。

```
redis-trib.rb add-node
```

其实内部就是发送了一个 gossip meet 消息给新加入的节点，通知那个节点去加入我们的集群。

- ping：每个节点都会频繁给其它节点发送 ping，其中包含自己的状态还有自己维护的集群元数据，互相通过 ping 交换元数据。
- pong：返回 ping 和 meeet，包含自己的状态和其它信息，也用于信息广播和更新。
- fail：某个节点判断另一个节点 fail 之后，就发送 fail 给其它节点，通知其它节点说，某个节点宕机啦。

#### ping 消息深入

ping 时要携带一些元数据，如果很频繁，可能会加重网络负担。

每个节点每秒会执行 10 次 ping，每次会选择 5 个最久没有通信的其它节点。当然如果发现某个节点通信延时达到了 `cluster_node_timeout / 2`，那么立即发送 ping，避免数据交换延时过长，落后的时间太长了。比如说，两个节点之间都 10 分钟没有交换数据了，那么整个集群处于严重的元数据不一致的情况，就会有问题。所以 `cluster_node_timeout` 可以调节，如果调得比较大，那么会降低 ping 的频率。

每次 ping，会带上自己节点的信息，还有就是带上 1/10 其它节点的信息，发送出去，进行交换。至少包含 `3` 个其它节点的信息，最多包含 `总节点数减 2` 个其它节点的信息。

### 分布式寻址算法

- hash 算法（大量缓存重建）
- 一致性 hash 算法（自动缓存迁移）+ 虚拟节点（自动负载均衡）
- redis cluster 的 hash slot 算法

#### hash 算法

来了一个 key，首先计算 hash 值，然后对节点数取模。然后打在不同的 master 节点上。一旦某一个 master 节点宕机，所有请求过来，都会基于最新的剩余 master 节点数去取模，尝试去取数据。这会导致**大部分的请求过来，全部无法拿到有效的缓存**，导致大量的流量涌入数据库。

 [![](D:\File\studyNote\images\hash.png)]()

#### 一致性 hash 算法

一致性 hash 算法将整个 hash 值空间组织成一个虚拟的圆环，整个空间按顺时针方向组织，下一步将各个 master 节点（使用服务器的 ip 或主机名）进行 hash。这样就能确定每个节点在其哈希环上的位置。

来了一个 key，首先计算 hash 值，并确定此数据在环上的位置，从此位置沿环**顺时针“行走”**，遇到的第一个 master 节点就是 key 所在位置。

在一致性哈希算法中，如果一个节点挂了，受影响的数据仅仅是此节点到环空间前一个节点（沿着逆时针方向行走遇到的第一个节点）之间的数据，其它不受影响。增加一个节点也同理。

燃鹅，一致性哈希算法在节点太少时，容易因为节点分布不均匀而造成**缓存热点**的问题。为了解决这种热点问题，一致性 hash 算法引入了虚拟节点机制，即对每一个节点计算多个 hash，每个计算结果位置都放置一个虚拟节点。这样就实现了数据的均匀分布，负载均衡。

![](D:\File\studyNote\images\consistent-hashing-algorithm.png)

#### redis cluster 的 hash slot 算法

redis cluster 有固定的 `16384` 个 hash slot，对每个 `key` 计算 `CRC16` 值，然后对 `16384` 取模，可以获取 key 对应的 hash slot。

redis cluster 中每个 master 都会持有部分 slot，比如有 3 个 master，那么可能每个 master 持有 5000 多个 hash slot。hash slot 让 node 的增加和移除很简单，增加一个 master，就将其他 master 的 hash slot 移动部分过去，减少一个 master，就将它的 hash slot 移动到其他 master 上去。移动 hash slot 的成本是非常低的。客户端的 api，可以对指定的数据，让他们走同一个 hash slot，通过 `hash tag` 来实现。

任何一台机器宕机，另外两个节点，不影响的。因为 key 找的是 hash slot，不是机器。

![](D:\File\studyNote\images\hash-slot.png)

### redis cluster 的高可用与主备切换原理

redis cluster 的高可用的原理，几乎跟哨兵是类似的。

#### 判断节点宕机

如果一个节点认为另外一个节点宕机，那么就是 `pfail`，**主观宕机**。如果多个节点都认为另外一个节点宕机了，那么就是 `fail`，**客观宕机**，跟哨兵的原理几乎一样，sdown，odown。

在 `cluster-node-timeout` 内，某个节点一直没有返回 `pong`，那么就被认为 `pfail`。

如果一个节点认为某个节点 `pfail` 了，那么会在 `gossip ping` 消息中，`ping` 给其他节点，如果**超过半数**的节点都认为 `pfail` 了，那么就会变成 `fail`。

#### 从节点过滤

对宕机的 master node，从其所有的 slave node 中，选择一个切换成 master node。

检查每个 slave node 与 master node 断开连接的时间，如果超过了 `cluster-node-timeout * cluster-slave-validity-factor`，那么就**没有资格**切换成 `master`。

#### 从节点选举

每个从节点，都根据自己对 master 复制数据的 offset，来设置一个选举时间，offset 越大（复制数据越多）的从节点，选举时间越靠前，优先进行选举。

所有的 master node 开始 slave 选举投票，给要进行选举的 slave 进行投票，如果大部分 master node`（N/2 + 1）`都投票给了某个从节点，那么选举通过，那个从节点可以切换成 master。

从节点执行主备切换，从节点切换为主节点。

#### 与哨兵比较

整个流程跟哨兵相比，非常类似，所以说，redis cluster 功能强大，直接集成了 replication 和 sentinel 的功能。



## 2、redis cluster的搭建

### 2.1、redis cluster 的重要配置

`cluster-enabled <yes/no>`

`cluster-config-file <filename>`: 指定一个文件，保存 redis 实例的集群状态， 包括集群其他机器的信息，比如节点的上线和下线，故障转移等

`cluster-node-timeout <milliseconds>`: 节点存活超时时长，超过指定时长，认为节点宕机，master 宕机的话就会触发主备切换，slave 宕机就不会提供服务

### 2.2、在三台机器上启动6个 redis 实例

redis cluster 要求至少3个 master，去组成一个高可用，健壮的分布式的集群，每个 master 都建议至少给一个 slave，即至少有 3个 master，3个 slave

在正式的环境下，建议在6台机器上去搭建。

下面开始 redis cluster 的搭建

```shell
mkdir -p /etc/redis-cluster   #存放redis 实例的集群状态文件
mkdir -p /var/log/redis		  #存放日志文件	
mkdir -p /var/redis/7001	  #存在持久化文件

#修改配置文件
# 7001.conf  
port 7001
cluster-enabled yes
cluster-config-file /etc/redis-cluster/node-7001.conf
cluster-node-timeout 15000
daemonize	yes							
pidfile		/var/run/redis_7001.pid 						
dir 		/var/redis/7001		
logfile /var/log/redis/7001.log
bind 192.168.31.187		
appendonly yes
```

至少要用3个 master 节点启动，每个 master 加上一个 slave 节点，先选择 6 个节点，启动 6 个实例

准备上面的配置文件 6份，分别为：7001.conf，7002.conf， 7003.conf，7004conf， 7005.conf，7006.conf，

因为我们这里只有3台机器，所以在每台机器上分别放两个配置文件置 /etc/redis 目录下

### 2.3、准备生产环境的启动脚本

在/etc/init.d 下，放 6 个启动脚本，分别为 ： redis_7001，redis_7002...

每个启动脚本内，都修改对应的端口号

### 2.4、分别在3台机器上，启动6个redis实例

将每个配置文件中的slaveof给删除

```shell
cd /etc/init.d/
./redis_7001 start  #启动redis实例
```

### 2.5、创建集群

```shell
#准备环境
yum install -y ruby
yum install -y rubygems
gem install redis

cp /usr/local/redis-3.2.8/src/redis-trib.rb /usr/local/bin

redis-trib.rb create --replicas 1 10.116.9.1:7001 10.116.9.1:7002 10.116.9.2:7003 10.116.9.2:7004 10.116.9.33:7005 10.116.9.33:7006

--replicas: 指定master有几个slave

redis-trib.rb check 10.116.9.1:7001
```

### 2.6、读写分离+高可用+多master

读写分离：每个 master 都有一个 slave

高可用：master 宕机，slave 自动被切换过去

多master：横向扩容，支持更大的数据量

## 3、redis cluster的常用命令

### 3.1、加入新的 master 节点

```shell
redis-trib.rb add-node 10.116.9.1:7007 10.116.9.1:7001 
#将新的redis实例连接到集群中 第一个是新增的节点，第二个可以是集群中的任意一个节点
```

新增加的 master 节点是没有 slots 的，无法处理任何数据，所以需要进行重新分配

```shell
redis-trib.rb reshard 10.116.9.1:7001
How many slots do you want to move (from 1 to 16384)? 4096
What is the receiving node ID?  03ccad2ba5dd1e062464bc7590400441fafb63f2 #新节点node id 
Please enter all the source node IDs. 
 Type 'all' to use all the nodes as source nodes for the hash slots. 
 Type 'done' once you entered all the source nodes IDs. 
Source node #1: 之前3个master 的 node id
Source node #2: 之前3个master 的 node id
Source node #3: 之前3个master 的 node id
Source node #4: done
```

把之前 3 个 master 上，总共 4096 个 hashslot 迁移到新加入的 master 节点上

### 3.2、添加新的 slave 节点

```shell
redis-trib.rb add-node --slave --master-id 03ccad2ba5dd1e062464bc7590400441fafb63f2 10.116.9.1:7008 10.116.9.1:7001
# --slave 表示添加的是 slave 节点
# -- master-id 加入到的master的node id
#10.116.9.1:7008 新节点
#10.116.9.1:7001 集群中任意节点
```

### 3.3、删除节点

**1、删除从节点**

```shell
redis-trib.rb del-node 10.116.9.1:7008 '9c240333476469e8e2c8e80b089c48f389827265' 
```

**2、删除主节点**

- 如果主节点有从节点，先将从节点转移到其他主节点

```shell
#将7002 转移到其他 master
redis-cli -h 10.116.9.1 -p 7002
10.116.9.1:7002> cluster replicate 0d4d3e1102d8d65454f33d2b37febe7a64b6d2c7 #master node id 
OK
```

- 如果主节点有 slot，先 reshard，然后再删除主节点

#### 4、slave的自动迁移

比如现在有10个master，每个有1个slave，然后新增了3个slave作为冗余，有的master就有2个slave了，有的master出现了salve冗余

如果某个master的slave挂了，那么redis cluster会自动迁移一个冗余的slave给那个master

只要多加一些冗余的slave就可以了

为了避免的场景，就是说，如果你每个master只有一个slave，万一说一个slave死了，然后很快，master也死了，那可用性还是降低了

但是如果你给整个集群挂载了一些冗余slave，那么某个master的slave死了，冗余的slave会被自动迁移过去，作为master的新slave，此时即使那个master也死了

还是有一个slave会切换成master的

之前有一个master是有冗余slave的，直接让其他master其中的一个slave死掉，然后看有冗余slave会不会自动挂载到那个master







