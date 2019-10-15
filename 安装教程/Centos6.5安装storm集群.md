## 1、Centos6.5配置storm集群

（1）安装 Java 7 和 Python 2.6.6

（2）下载并解压 storm 安装包

（3）目录重命名  

`mv apache-storm-1.1.0/ storm/`

（4）配置环境变量

```shell
cd /usr/local
vi ~/.bashrc
#添加下面配置
export  STORM_HOME=/usr/local/storm
export PATH=/usr/local/sbin:/usr/local/bin:/sbin/:/bin:/usr/bin:/usr/sbin:/root/bin:$ZOOKEEPER_HONE/bin:$SCALA_HOME/bin:$STORM_HOME/bin:$PATH

source ~/.bashrc
```

（5）修改 storm 配置文件

```shell
#创建storm本地目录
mkdir /var/storm 
vi usr/local/storm/conf/storm.yaml
#修改
storm.zookeeper.servers:
    - "10.116.9.1"
    - "10.116.9.2"
    - "10.116.9.33"

nimbus.seeds: ["10.116.9.1"]

#添加
storm.local.dir: "/var/storm"
#指定每台机器上可以启动的worker数量，一个端口号代表一个worker
supervisor.slots.ports:
    - 6700
    - 6701
    - 6702
    - 6703
```

将上述配置拷贝到其他两台机器上，部署集

```shell
cd /usr/local
scp ~/.bashrc root@eshop-cache02:~/
scp ~/.bashrc root@eshop-cache03:~/
scp -r storm root@eshop-cache02:/usr/local/
scp -r storm root@eshop-cache03:/usr/local

#在其他两台机器上创建storm本地目录
mkdir /var/storm
source ~/.bashrc
```

启动 storm 集群和 ui 界面，注意**启动之前要先运行zookeeper集群**

```shell
#配置的nimbus节点
storm nimbus >/dev/null 2>&1 &
#所有节点启动supervisor
storm supervisor >/dev/null 2>&1 &
#nimbus节点上启动ui界面，访问端口号为8080
storm ui >/dev/null 2>&1 &

#可以使用jps命令查看启动是否成功

```



