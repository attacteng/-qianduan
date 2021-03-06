[TOC]

# 1、高并发基本概述
## 1.1 高可用
一个系统经过专门的设计，从而减少停工时间，而保持其服务的高度可用性。
## 1.2 高并发
 - 高并发是互联网分布式系统架构设计中必须考虑的因素之一，它通常是指，通过设计保证系统能够同时并行处理很多请求。
 - 并发相关的指标有：响应时间（response time），吞吐量（troughput），每秒查询率QPS（query per second），并发用户等。
 - 响应时间，系统请求作出的响应时间。例如系统处理一个HTTP请求需要200ms，这就是系统的响应时间。
 - 吞吐量，单位时间内处理的请求数量。
 - 每秒查询QPS，每秒响应请求数。
 - 并发用户数。

# 2、Redis的过期策略和内存淘汰策略
## 2.1 过期策略
定时过期：
 - 每个设置过期时间的key都需要创建一个定时器，到过期时间就会立即清除。该策略可以立即清除过期的数据，对内存很友好；但是会占用大量的CPU资源去处理过期的数据，从而影响缓存的响应时间和吞吐量。
惰性过期：
 - 只有当访问一个key时，才会判断该key是否已过期，过期则清除。该策略可以最大化地节省CPU资源，却对内存非常不友好。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。
定期过期：
 - 每隔一定的时间，会扫描一定数量的数据库的expires字典中一定数量的key，并清除其中已过期的key。该策略是前两者的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时，可以在不同情况下使得CPU和内存资源达到最优的平衡效果。
Redis中同时使用了惰性过期和定期过期两种过期策略：
 - 所谓定期删除，指的是redis默认每隔100ms就随机抽取一些设置了过期时间的key，检查其是否过期，如果过期就删除。
 - 假设redis里放了10W个key，都设置了过期时间，你每隔几百毫秒，就检查10W个key，那redis基本上就死了，cpu负载会很高的，消耗在你的检查过期key上了。注意，这里可不是每隔100ms就遍历所有的设置过期时间的key，哪有就是一场性能上的灾难。实际上redis是每隔100ms随机抽取一些Key来检查和删除的。
 - 但是，定期删除可能会导致很多过期Key到了时间并没有被删除掉。所以就是惰性删除了。也就是说，在你获取某个key的时候，redis会检查一下，这个key如果设置了过期时间那么是否过期了？如果过期了此时就会删除，不会返回任何东西。
 - 获取Key的时候，如果此时key已经过期，就删除，不会返回任何东西。
 - 实际上有可能，如果定期删除漏掉了很多过期key，然后也没及时去查，也就没惰性删除，此时，可能会有大量过期key堆积在内存里，导致redis内存块耗尽了。只能走内存淘汰机制来解决。

## 2.2 内存淘汰机制
Redis的内存淘汰策略是指在Redis的用于缓存的内存不足时，怎么处理需要新写入且需要申请额外的空间的数据。
 - noeviction  当内存不足以容纳新写入数据时，新写入操作会报错。
 - allkeys-lru  当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key。
 - allkeys-random  当内存不足以容纳写入新数据时，在键空间中，随机移除某个key。
 - volatile-lru  当内存不足以容纳写入新数据时，在设置了过期时间的键空间中，移除最近最少使用的key。
 - valatile-random  当内存不足以容纳写入新数据时，在设置了过期时间的键空间中，随机移除某个Key。
 - valatile-ttl  当内存不足以容纳写入新数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除。

