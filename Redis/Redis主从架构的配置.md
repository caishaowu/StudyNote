## 配置Redis主从架构

### 1.1、修改 redis.conf 文件

将 `slaveof`功能打开，配置上对应的ip和端口

```shell
slaveof 10.116.10.223 6379
```

也可使用`slaveof`命令

### 1.2、强制读写分离

打开 `slave-read-only`，默认情况下是开启的

开启了之后，redis 会拒绝所有的写操作，这样可以强制搭建成读写分离的架构

### 1.3、集群安全认证

master上启用安全认证，`requirepass`
master连接口令，`masterauth`

### 1.4、修改 bind 配置

在搭建生产环境集群的时候，不要忘记修改一个配置，bind

默认的配置为 `bind 127.0.0.1` ，这是 本地的开发调试环境，只能本地才能访问到6379端口

所以我们要在每个redis中，在 redis.conf 文件中将其修改为 `bind 自己的ip地址`

然后再让防火墙开放对6379端口的访问 `iptables -A INPUT -ptcp --dport  6379 -j ACCEPT`

> 修改 bind 配置之后 进入redis不能直接使用 redis-cli 命令了，要在后面加上ip地址
>
> 例：redis-cli -h 10.116.10.223

至此，Redis的主从架构搭建完成。