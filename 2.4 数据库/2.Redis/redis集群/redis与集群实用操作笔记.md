- [redis与集群实用操作笔记](https://www.cnblogs.com/a-News/p/15850522.html)

# redis哨兵

## 部署方式

### redis配置

首先需要区分的是主从redis，主机也就是用来写的机器，从机是从来读的，为主机分担压力，与集群不同的是redis哨兵**不可通过从机写入数据同步到主机，但是也可以配置参数实现从机可写**

```
你可以配置salve实例是否接受写操作。可写的slave实例可能对存储临时数据比较有用(因为写入salve的数据在同master同步之后将很容易被删除）

slave-read-only yes
redis主机还是和正常的redis配置一样

redis从机需要配置一下参数（192.168.6.136 16379是主redis地址）

slaveof 192.168.6.136 16379 # redis旧版本使用

replicaof 192.168.6.136 16379 # redis 5.x版本往后更新
slave-priority 数值

值必须满足>0 ，默认100，就是使用哨兵时，当主master，哨兵会优先选择slave-priority数值小的当主机
```

redis不用多说，主从机器配置好后就可以正常运行了，这时候从机就会主动同步主机的内容

### 哨兵配置

首先说下哨兵是干啥的

**如果没有哨兵，当主机失联之后，人为的知道了他失联了，需要手动把一台redis切换为主redis，这就需要人工干预，费事费力，而且还会导致服务一段时间不可用**

**哨兵切换**：你配置的哨兵都是独立运行的进程，它们都是自己独立运行，不用依赖于redis，实现切换主从的原理就是，首先主机宕机了，哨兵会定时向主节点发送ping命令，然后哨兵中可以配置一个参数**down-after-milliseconds，如果超过down-after-milliseconds**还未收到某节点回复，则认为该节点下线，仅仅一个哨兵发现主机下线并不会对主机进行操作，需要等一定的哨兵认定他下线后（哨兵配置的主机地址后面的数值参数），**哨兵之间会互相检测（每两秒交换一次数据）**才进行操作，当哨兵检测满足了切换主机的条件后，将会进行一次投票，投票通过得主机获得了主机权限，并且哨兵会自动更改redis与哨兵的配置文件，死掉的主机恢复后，也会自动切换到此机器下成为此机器的从机，哨兵配置http://wiki.sdnetwan.net/pages/viewpage.action?pageId=28610094

**注意的点**，哨兵配置参数**down-after-milliseconds**的默认值是30000，即30s，可以根据不同的网络环境和应用要求来调整：值越大，对主观下线的判定会越宽松，好处是误判的可能性小，坏处是故障发现和故障转移的时间变长，客户端等待的时间也会变长。例如，如果应用对可用性要求较高，则可以将值适当调小，当故障发生时尽快完成转移；如果网络环境相对较差，可以适当提高该阈值，避免频繁误判。

**sentinel failover-timeout**与故障转移超时的判断有关，但是该参数不是用来判断整个故障转移阶段的超时，而是其几个子阶段的超时，例如如果主节点晋升从节点时间超过这个时间，或从节点向新的主节点发起复制操作的时间(不包括复制数据的时间)超过这个时间，都会导致故障转移超时失败。

failover-timeout的默认值是180000，即180s；如果超时，则下一次该值会变为原来的2倍。

哨兵坑：配置下面的参数时与哨兵切换主从投票的坑

**sentinel monitor**    

**注意！！！！ 此处的quorum只是代表了客观上主节点失联，选举主redis时还需要通过半数以上的投票才可**

```
配置：

protected-mode no
bind 0.0.0.0
port 16380
daemonize yes
sentinel monitor master 192.168.6.136 16379 2
sentinel down-after-milliseconds master 5000
sentinel failover-timeout master 180000
sentinel parallel-syncs master 1
sentinel auth-pass master 123456
logfile /var/log/redis/sentinel-16380.log



配置完生成：增加的配置地址为自动获取的从机地址

protected-mode no
bind 0.0.0.0
port 26380
daemonize yes
sentinel myid 2f296b552047f4f3a51e76807b5242d816303afd
sentinel deny-scripts-reconfig yes
sentinel monitor master 192.168.6.136 16379 2

sentinel down-after-milliseconds master 15000
logfile "/var/log/redis/sentinel-26380.log"

# Generated by CONFIG REWRITE
dir "/usr/local/redis-sentinel"
sentinel auth-pass master 123456
sentinel config-epoch master 437
sentinel leader-epoch master 437
sentinel known-replica master 192.168.6.137 16379
sentinel known-replica master 192.168.6.136 26379
sentinel known-sentinel master 192.168.6.136 26380 16b5148c5b96e169f1a4f3023a26d1df0c8abf81
sentinel known-sentinel master 192.168.6.137 16380 918a0c444822bbf5612fed5d181a14d203b9f2df
sentinel current-epoch 437
```

## 哨兵的三个定时任务

- 定时任务一：每个哨兵节点每10 秒会向主节点和从节点发送info 命令获取最拓扑结构图，哨兵配置时只要配置对主节点的监控即可，通过向主节点发送info，获取从节点的信息，并当有新的从节点加入时可以马上感知到 。
- 定时任务二：每个哨兵节点每隔2 秒会向redis  数据节点的指定频道上发送该哨兵节点对于主节点的判断以及当前哨兵节点的信息，同时每个哨兵节点也会订阅该频道，来了解其它哨兵节点的信息及对主节点的判断，其实就是通过消息publish 和subscribe 来完成的。
- 定时任务三：每隔1 秒每个哨兵会向主节点、从节点及其余哨兵节点发送一次ping 命令做一次心跳检测，这个也是哨兵用来判断节点是否正常的重要依据

想法：使用redis不改变单例的模式时可使用一个函数函数是一个生成器，每次都返回连接主机

## 哨兵实现故障转移选举新master的步骤

- 步骤一：在从节点中选择新的主节点：选择的原则是，首先过滤掉不健康的从节点；然后选择优先级最高的从节点(由redis中的参数slave-priority指定)；如果优先级无法区分，则选择复制偏移量最大的从节点；如果仍无法区分，则选择runid最小的从节点。
- 步骤二：更新主从状态：通过slaveof no one命令，让选出来的从节点成为主节点；并通过slaveof命令让其他节点成为其从节点。
- 步骤三：将已经下线的主节点(即6379)设置为新的主节点的从节点，当6379重新上线后，它会成为新的主节点的从节点。
- 走完上边三步骤后Redis的故障转移才算完整结束

# redis集群

Redis Cluster 集群模式具有 **高可用**、**可扩展性**、**分布式**、**容错** 等特性，用的CRC16算法，这是是一种循环校验算法

## 部署方式

### redis配置

```
cluster-enabled yes（启动集群模式）
cluster-config-file nodes.conf  集群中各个redis都有这个文件，保存的是所有node信息
cluster-node-timeout 15000   超时时限，就是集群中的redis互相检测，进行ping命令时超过这个时间就认定他失效了，默认15000

启动集群模式时，需要打开这几项的注释，其余是可不变
```

### 创建集群

```
redis-cli --cluster create 183.2.212.156:7001 183.2.212.156:7002 183.2.212.156:7006  172.30.10.46:7001 172.30.10.46:7002 172.30.10.46:7006 --cluster-replicas 1

--cluster-replicas 1 这个指定一主一从模式，前面的三个地址是主机，后面的三个地址是从机
```

### 集群fail认定条件

1. 当一半以上的主机同时宕机，集群fail了，不可用了
2. 当一个主机死掉，并且这个主机下面没有从节点来代替他的位置，则集群fail了，不可用了

### 集群切换主从

与哨兵差不多，不过不是哨兵进行故障的切换操作，是redis相互之间每一秒会进行通信，如果返回pong或者超过咱们设置的**cluster-node-timeout**参数后，则认定此主机宕机了，他也会有一个投票机制，当一个认为他宕机时，并不会进行故障转移，只有当认定他**死机的主机超过总主机的一半后**，才进行故障转移，会找此主机下面的活跃的slave代替主机位置，将主机槽位转移到此机器上

## 集群底层命令

坑：**redis集群的主从节点，不要在同一服务器上，这会导致一套服务器宕机，所有的机器都不可用！！**

### 集群

```
# 集群
cluster info   打印机群信息

cluster nodes 列出集群当前已知的所有节点，以及这些节点的相关信息
```

#### 节点操作

```
cluster meet <ip> <port> 将这个ip和port指定的节点增加到咱们的集群中

cluster forget <node_id> 将这个node移除（经过测试，假移除，只是假删除了1分钟，1分钟后，如果这个节点是可用的，则恢复），可用于切换主从

cluster replicate <node_id> 将此节点设置为该node记得从节点（注意，此节点必须为空节点，或者从节点）

cluster failover 可理解为切换主从的，人工判定主机的死亡，将此机器切换为主机
```

### 键操作

```
cluster keyslot <key>  计算键 key 应该被放置在哪个槽上

cluster countkeysinslot <slot>  返回槽 slot 目前包含的键值对数量

cluster getkeysinslot <slot> <count>  返回 count 个 slot 槽中的键  
```

## redis-cli集群命令

### 添加节点

```
# 添加节点
redis-cli  --cluster add-node 172.30.12.46:7003 172.30.12.46:7004 

前面的地址为集群中任意节点，后面是添加的节点地址


# 添加节点并且设置为某节点的从机
redis-cli --cluster add-node 172.30.12.46:7003 172.30.12.46:7004 --cluster-slave --cluster-master-id <node_id>

第一个地址为集群中任意节点，后面地址为添加的节点地址   最后node_id为需要绑定到主机的nodeid
```

### 移除节点

```
redis-cli  --cluster del-node 172.30.12.46:7003 <node_id>

前面的地址为集群中任意节点，后面是需要删除的node的node_id
```

### 集群

```
redis-cli --cluster check host:port  # 检查集群

redis-cli --cluster info host:port   # 查看集群状态

redis-cli --cluster fix host:port   #修复集群
redis-cli --cluster fix host:port --cluster-search-multiple-owners #修复槽的重复分配问题


redis-cli --cluster reshard host:post # 槽位分配
redis-cli --cluster reshard $cluster_ip --cluster-from $in_node --cluster-to $to_node --cluster-slots $slots --cluster-yes   # 分配槽位简单命令
docker_name=$1 # docker名
cluster_ip=$2 # 集群中的节点ip
to_node=$3  # 获取槽位的主机地址
in_node=$4  # 分配槽位的主机地址
slots=$5    # 槽位个数


redis-cli import host:port   #将外部redis数据导入集群
--cluster-from <arg>         #将指定实例的数据导入到集群
--cluster-copy               #migrate时指定copy
--cluster-replace            #migrate时指定replace 
```

坑：**当一个节点没有槽位时，无法转换主从，不能用failover转换主从！！！！！！！！！**