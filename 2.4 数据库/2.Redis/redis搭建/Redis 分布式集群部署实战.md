[TOC]

原理：

- Redis集群采用一致性哈希槽的方式将集群中每个主节点都分配一定的哈希槽，对写入的数据进行哈希后分配到某个主节点进行存储。
- 集群使用公式（CRC16 key）& 16384计算键key数据那个槽。
- 16384个slot均匀分布在各个节点上。
- 集群中每个主节点将承担一部分槽点的维护，而槽点中存储着数据，每个主节点都有至少一个从节点用于高可用。

节点通信方式：

- 开启一个端口 设置的端口号+10000，用于集群之间节点通信交换信息。
- 每个节点默认每秒10次选择随机5个节点发送ping消息，将自身信息和知道的集群信息传递，收到ping消息后返回pong消息做回复，最后通过这种随机的消息交换，最终每个节点将获得所有信息。
- 当某个主节点挂掉，所有节点将会发现主节点挂掉了，作为主节点的从节点，就会接替主节点的工作，然后告诉所有其它节点，他成为了主。这样其它存活节点，就将它们维护的信息表更新从节点将接任做主，如果都挂掉集群将报错。当从一个节点操作，根据一致性哈希计算后将存储在其中一个主节点中，从节点将同步主的数据。

![](https://img2020.cnblogs.com/blog/1024722/202007/1024722-20200703125740642-1853682721.png)

- redis cluster是去中心化的，集群中的每个节点都是平等的关系，每个节点都保存各自的数据和整个集群的状态。每个节点都和其他所有节点连接，而且这些连接保持活跃。
- 搭建集群时，会为每一个分片的主节点，对应一个从节点。实现slaveof功能，同时当主节点down，实现sentinel哨兵的自动failover切换功能.
