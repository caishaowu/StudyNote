## 1、zookeeper集群搭建

1. 将下载好的 zookeeper-3.4.5.tar.gz 安装包拷贝到 /usr/local 目录下
2. tar -zxvf zookeeper-3.4.5.tar.gz
3. 重命名目录： mv zookeeper-3.4.5 zk
4. 配置 zk 相关的环境变量及配置文件

```shell
vi ~/.bashrc
export ZOOKEEPER_HONE=/usr/local/zk
export PATH=/bin:/usr/bin:$ZOOKEEPER_HONE/bin:$PATH
source ~/.bashrc

cd zk/conf
cp zoo_sample.cfg zoo.cfg
vi zoo.cfg
#修改
dataDir=/usr/local/zk/data
#新增
server.0=eshop-cache01:2888:3888	
server.1=eshop-cache02:2888:3888
server.2=eshop-cache03:2888:3888

mkdir data
cd data
vi myid
0
```

在另外两个节点上按照上述步骤配置ZooKeeper，使用scp将zk和.bashrc拷贝到eshop-cache02和eshop-cache03上，然后source一下即可。唯一的区别是标识号分别设置为1和2。

```shell
scp ~/.bashrc root@eshop-cache02:~/
scp ~/.bashrc root@eshop-cache03:~/
scp -r /usr/local/zk root@eshop-cache02:/usr/local/
scp -r /usr/local/zk root@eshop-cache03:/usr/local/

#分别在三台机器上执行启动命令
zkServer.sh start
#查看zookeeper状态,应该有一个leader，两个follower
zkServer.sh status
```

## 2、kafka集群搭建

### 2.1、配置scala

1. 将下载好的 scala-2.11.4.tgz 安装包拷贝到 /usr/local 目录下
2. tar -zxvf scala-2.11.4.tgz 
3. 重命名目录：mv scala-2.11.4 scala
4. 配置 scala 相关的环境变量

```shell
vi ~/.bashrc
export SCALA_HOME=/usr/local/scala
export PATH=:/bin:usr/bin:$ZOOKEEPER_HONE/bin:$SCALA_HOME/bin:$PATH
#查看是否安装成功
scala -version
```

在另外两个节点上按照上述步骤配置 scala，使用scp将 scala 和.bashrc拷贝到eshop-cache02和eshop-cache03上，然后source一下即可。

```shell
scp ~/.bashrc root@eshop-cache02:~/
scp ~/.bashrc root@eshop-cache03:~/
scp -r /usr/local/scala root@eshop-cache02:/usr/local/
scp -r /usr/local/scala root@eshop-cache03:/usr/local/
```

### 2.2、配置kafka

1. 将下载好的 kafka_2.11-0.10.0.0.tgz 安装包拷贝到 /usr/local 目录下
2. tar -zxvf kafka_2.11-0.10.0.0.tgz
3. 重命名目录：mv kafka_2.11-0.10.0.0 kafka
4. 配置 kafka 

```shell
vi /usr/local/kafka/config/server.properties
broker.id：依次增长的整数，0、1、2，集群中Broker的唯一id
zookeeper.connect=10.116.9.1:2181,10.116.9.2:2181,10.116.9.33:2181
```

**注意：在与SpringBoot整合时，kafka-clients的版本要与安装的kafka版本一致，否则会报错。**

```xml
<dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
            <version>1.1.1.RELEASE</version>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.kafka</groupId>
                    <artifactId>>kafka-clients</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>0.10.0.0</version>
        </dependency>
```



### 2.3、安装slf4j

1. 将下载好的slf4j-1.7.6.zip 安装包拷贝到 /usr/local 目录下
2. unzip slf4j-1.7.6.zip
3. cp slf4j-nop-1.7.6.jar /usr/local/kafka/libs/
4. 解决kafka Unrecognized VM option 'UseCompressedOops'问题

```shell
vi /usr/local/kafka/bin/kafka-run-class.sh
if [ -z "$KAFKA_JVM_PERFORMANCE_OPTS" ]; then
  KAFKA_JVM_PERFORMANCE_OPTS="-server  -XX:+UseCompressedOops -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:+CMSScavengeBeforeRemark -XX:+DisableExplicitGC -Djava.awt.headless=true"
fi

#去掉-XX:+UseCompressedOops即可
```

按照上述步骤在另外两台机器分别安装kafka。用scp把kafka拷贝到其他机器即可。
唯一区别的，就是server.properties中的broker.id，要设置为1和2

```shell
scp -r /usr/local/kafka root@eshop-cache02:/usr/local/
scp -r /usr/local/kafka root@eshop-cache03:/usr/local/
scp -r /usr/local/slf4j-1.7.6 root@eshop-cache02:/usr/local/
scp -r /usr/local/slf4j-1.7.6 root@eshop-cache03:/usr/local/
#在三台机器上的kafka目录下，分别执行以下命令
nohup bin/kafka-server-start.sh config/server.properties &
#nohup: 忽略输入并把输出追加到"nohup.out" 按回车即可，忽略
#使用jps检查启动是否成功
jps

#使用基本命令检查kafka是否搭建成功

bin/kafka-topics.sh --zookeeper 10.116.9.1:2181,10.116.9.2:2181,10.116.9.33:2181 --topic productInfoService --replication-factor 1 --partitions 1 --create

#执行该命令后，命令行进入等待状态，等待用户输入消息
bin/kafka-console-producer.sh --broker-list 10.116.9.1:9092,10.116.9.2:9092,10.116.9.33:9092 --topic productInfoService
#执行该命令后，命令行进入等待状态，等待接收用户输入的消息
bin/kafka-console-consumer.sh --zookeeper 10.116.9.1:2181,10.116.9.2:2181,10.116.9.33:2181 --topic productInfoService --from-beginning
```

