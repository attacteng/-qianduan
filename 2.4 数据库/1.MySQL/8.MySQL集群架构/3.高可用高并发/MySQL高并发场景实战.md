[MySQL高并发场景实战](https://developer.aliyun.com/article/783099?spm=a2c6h.13528211.0.0.228c4307XujuTp)



# 一、系统调优

从六个方面进行：容量评估、性能评测、架构调优、实例调优、内核调优和监控报警。

**（一）容量评估**

容量评估主要分为三个部分：经验评估、单元压测与全链路压测。



l  **经验评估**

容量评估刚开始阶段是经验评估，根据以往经验值给出一个预估的压力，再除以单台机器的性能，大致可得出所需服务器的数量。根据服务器数量、应用机器数量与DB机器数量，可以得出整个数据库（如链接池）的设置，以及要扩充多少实例和应用机器等，判断能否支撑得住双11的容量。需要针对上述的预估做一个判断验证，通过压测来完成。

l  **单元压测**

经过经验预估后，需要针对上述的预估做判断验证，可以通过单元压测来完成。由于是分布式的系统，需要针对单个实例、单个单元以及单个应用模块来进行压测。单元压测的主要功能是完成单元内的验证，系统整个的架构很多是异地多国，因此不但需要验证整个双11的流量，还要验证在单元内的容量是否充足，以及单个系统的容量，例如压测某一个模块、交易模块、优惠模块等容量是否充足。

l  **全链路压测**

等每一个应用模块验证完成之后，需要对整个链路进行压测，也就是全链路压测。全链路压测是基于场景化的仿真测试，其数据最接近业务的系统值，针对全链路压测，可以借助压测来验证整个分布式系统的容量是否充足。

**（二）性能评测**

1.性能评测的意义

1）目的

发现基础设施瓶颈、中间件瓶颈、系统容量瓶颈；

发现分布式系统短板。

2）用途

保障大促容量充足，保障业务正常运转；

保障核心功能、保证用户体验；

评估大促成本，包含人工成本、机器成本、运维成本等。

3）难点

真实业务场景压测；

真实SQL模拟。

 

2.基准测试

压测可以分为基准测试和仿真测试。对于MySQL而言，基准测试可以用sysbench或mysqlslap等。基于通用场景的测试，每个实例在不同的业务场景下达到的容量也不同，通常情况下，同一个实例的应用真实的业务场景的值不会超过其基准测试的值。

3. 全链路压测是大促备战核武器

真实业务场景的压测往往比基准测试更加复杂，阿里巴巴在双11的场景下整个压测过程大概可以分为4个步骤，如下图所示：





首先针对涉及到的业务进行模块梳理与技术架构梳理，以及容量预估是否充足。梳理完成之后需要进行环境准备，包含服务器准备、数据构造与业务请求准备等。第三步正式压测，可以通过压测来发现短板，验证容量是否充足。预案验证可以验证在某个压力场景下，降级某个预案会给系统减少多少压力。验证压测执行完之后，需要针对过程中发现的短板、架构以及容量等问题进行优化，这个过程并非一蹴而就，而是要经过多轮压测。

阿里巴巴集团及各BU每年压测4000+次，13年全链路发现700+问题，14年发现500+问题，15年发现400+问题。真实业务场景的全链路压测，已成为每年双11前筹备工作的核心之一。



 

**4.****工具**

对于上述的全链路压测操作，我们有一些现成的工具以供使用。

1）PTS（Performance Testing Service）

是面向所有技术背景人员的云化测试工具，可理解为全链路压测工具。有别于传统工具的繁复，PTS以互联网化的交互，提供性能测试、API调试和监测等多种能力。自研和适配开源的功能可以轻松模拟任意体量的用户访问业务的场景，任务随时发起，免去繁琐的搭建和维护成本。紧密结合监控、流控等兄弟产品提供一站式高可用能力，高效检验和管理业务性能。

2）DAS（Database Autonomy Service）

智能压测主要应用在以下两种场景：

l  为应对即将到来的短期业务高峰，验证当前的RDS MySQL规格是否需要扩容；

l  数据库迁移上云前，验证目标RDS MySQL的规格是否满足业务需求。

**5.****注意事项**

根据以往的情况，性能评测时主要有4个方面需要特别注意：



1）参数

如果想对比一两个参数在不同情况下，例如打开和关闭、设置不同的值时对性能的影响，可以单独做测试。一旦测试出来最优值，需在压测之前将参数调到最优值。

2）网络

曾有用户拿线下的机器链接本地数据库，然后拿本地的应用机器链接RDS做性能对比，这种对比不准确，主要原因是压测机到RDS的网络延迟和到本地数据库的延迟存在很大差异。

3）规格

RDS的规格的性能差别较大,如果想测试CPU的性能，物理IO要少，数据大小在内存容量范围内即可。但大多数业务场景都是涉及到物理IO，所以在到测试数据时，数据量要大于内存的大小

4）ECS的网络带宽

阿里云的ECS是限制网络带宽的，以往有用户在做测试时，RDS的资源没有用满，压力也上不去,经过定位发现是ECS的网络带宽打满了，因此在准备整个压测环境时，要将这些内容调好。



**（三）架构调优**

**1.****数据库架构调优**

如果业务的访问都用数据库支撑的话成本高昂,缓存可以代替一部分关系型数据库在读方面的请求。基于原理的设计以及成本方面考虑，缓存的读性能比关系型数据库好，性价比较高。

到数据库层面，如果是读多写少，针对于单个实例很难支撑的情况下，可以借助于只读实例。只读实例可以实现在线弹性的扩展读能力，读的业务请求可以实现隔离，例如可以把轻分析型以及拖数据类型在只读实例内完成。

此外，每个只读实例都有一个单独的链接地址，如果把某一类的业务和其他的业务区分开，例如某一类的只读的这个场景，只到某一个实例访问，可以单独链接只读实例的链接串。

如果要是想从整个层面来控制主实例和只读实例的访问，可以借助负载均衡独享代理完成。独享代理可以缓解大量短链接的场景，使用代理后不用反复变更应用类的链接地址，减少维护成本。使用独享代理之后，可以对线上的资源实现可扩展，承受更高的流量。如果是RDS的实例规格以及只读实例都已经升到最大，但仍然不能支撑业务发展的话，可以考虑把RDS的升级到Polar Mysql或者是分布分表Polar X2.0，完成读写容量的扩展。

 

**2.** **降低只读实例和主实例的延迟**

针对主实例和只读实例的数据一致性的问题，例如有延迟以及中断场景。针对线上延迟的问题我们做了分析，得出原因主要包括5个方面：

**1****）主实例的DDL**

占40%，如Alter、Drop等。需要Kill DDL语句或用DMS的无锁变更等。

**2****）主实例的大事务**

占20%，如大批量导入、删除、更新数据。需要将大事务拆分成为小事务进行批量提交，这样只读节点就可以迅速的完成事务的执行，不会造成数据的延迟。

**3****）主实例写入压力过大**

占20%，如主实例规格较小、压力过大。这种情况需要升级主实例和只读实例的规格。

**4****）只读节点规格过小**

占10%，这种情况升级只读实例规格。

**5****）其他（无主键）**

占10%， RDS目前已经支持对表添加隐式主键，但是对于以前历史创建的表需要进行重建才能支持隐式主键。

 

**3.** **提升缓存命中率**

在高并发场景下，写入压力过大，如何提升缓存的命中率？

根据以往的经验，有4种更新方式：Cache aside，Read through，Write through，Write behind caching。

缓存的更新时，应用可以从Cache里面读取数据，取到后返回，没有得到从数据库里面取，成功后再返回缓存当中；

Read through是已读没有读到，就更新缓存；

Right through是在数据更新时，发现缓存中没有就更新缓存；

Write behind caching类似于底层的 Linux操作系统机制，如果要是用Write behind caching合并同一个数据的多次操作，以上几种方式都不能保证缓存命中率。

我们做核心系统时有一个小巧思，下面以产品的形式进行解释。



如上图所示，应用程序把写请求到RDS,读请求到 Redis。RDS实时获取变更数据，通过订阅数据变更的方式拿到增量数据后，更新Redis，这个巧思有以下好处：

l  **更新路径短，延迟低**

缓存失效为异步流程，业务更新DB完成后直接返回，不需要关心缓存失效流程， 整个更新路径短 ，更新延迟低。

l  **应用简单可靠**

应用无需实现复杂双写逻辑，只需启动异步线程监听增量数据，更新缓存数据即可。

l  **应用更新无额外性能消耗**

因为数据订阅是通过解析DB的增量日志来获取增量数据，获取数据的过程对业务、DB性能无损。



**（四）实例调优**

实例调优主要包含两部分：弹性扩容和参数调优。

**1.****弹性扩容**





**2.****参数调优**

参数调优主要从三个方面进行，分别问连接、内存和IO，如下图所示，这里重点阐述几个参数。



连接相关的参数，back_log是标记MySQL的并发连接数。如果在短时间内有大量的连接请求时，主线程会在一段时间内或者是瞬间检查连接数并启动新连接。

table_open_cache是所有线程打开表的数量，性能较小时会导致性能下降，但也不能设置过大，否则可能会造成 OMM。

IO相关的参数，sync_binlog大家不会陌生，我们在每年的双11会将这些数值调到最优。io_capacity和刷脏是IO两个比较重要的参数。

内存相关的主要参数是innodb_buffer_pool_instances。在5.6之前，相对来说innodb_buffer_pool_instances只有一个时，如果内存越大，它的性能越不稳定，因为刷脏的时候要锁。当有buffer_pool_instances了之后，会把Buffer拆到好几个内存的List中。这种情况下，每一个内存都会维护自己的List和锁结构，相当把锁拆分，性能会提升很多。此外还有和几个连接相关的参数，例如join_buffer以及临时表。

针对影响性能几个主要的参数，用户可以单独设置。这里我们已经将对性能影响较大的参数调到高性能模板中，应用时可以找到对应的参数，应用到对应的实例上即可。



上图是普通模板与高性能模板的一个对比，可以看到，相比高性能模板，普通模板的性能要低25%左右。

**（五）内核调优**

**1.****版本升级**



如上图所示，从版本上来看，目前官方维护的是5.6、5.7和8.0版本，每一个大版本的升级会带来一些新的特性，同时它的性能也会有上升，从表中可以看到8.0性能是最强的。但升级时也会遇到一些问题，例如执行计划有一些和原来版本不太兼容的地方。

**2.** **AliSQL****特性**

**1****）内容调优的4个补丁**



针对高并发与热点库存的场景，AliSQL提供了4个补丁解决，其中内存注释、语句队列以及语句返回这3个补丁是针对热点库存更新。

 

l  **库存注释**



在秒杀业务中，从逻辑上来说减库存包含两个场景：拍下减库存和付款减库存。

拍下减库存表示客户拍下时把库存减掉,业务逻辑较为简单。付款减库存表示客户付完款后减掉库存，业务逻辑较为复杂，不是一条语句能够完成的，而是在事务中完成。在事务中，行锁冲突本来就较为严重，因此需要提升事务的性能。

通常的情况下，释放事务、开启事务和结束事务都是由应用来完成，如果是应用处理得过慢，或是网络交互的时间过慢，或者是网络拥堵，就会增加行锁持有的时间。

库存注释是把持有行锁的时间行锁释放，交给数据库自己来做。使用排队和事务性Hint来控制并发控制、快速提交或者回滚事务，成功就提交，失败就回滚，将控制权交给MySQL内核可以减少行锁持有的时间，快速地结束事务，提升减库存的吞吐能力。

l  **语句队列**



MySQL分为引擎层和Server层，在高并发的场景下同一行的行锁会在Server层和引擎层会来回切换。

InnoDB的事务锁是最细粒度的行级锁，如果语句针对相同行进行并发操作，会导致冲突比较严重，AliSQL将冲突放在Server层进行排队。对于相同行的冲突，如果让它在一个桶内，会减少冲突检测的开销，进而减少引擎层和Server层上下文切换带来的消耗。

 

l  **语句返回**



语句返回即更新完之后直接把结果返回，这个特性在PG和Oracle中都有，叫Returning，在MySQL没有，AliSQL吸纳了这个特性。

如果在一个事务中更新完一行记录之后，应用再发请求Select这一条结果再返回给应用，会多一次网络交互。如果直接把Update的结果返回，就可以减少这一次网络交互。性能的提升可以节省多次网络交互和应用判断的时间，带来事务性能与应用性能的提升。

l  **线程池**



对于高并发场景，AliSQL使用线程池来解决。

在有大量线程进行并发访问时，线程池会自动调节并发的线程数量在合理范围内，避免线程池因为线程数量过多造成大量缓存失效。

线程池会将语句和事务分为不同的优先级，分别控制语句和事务的并发数量，减少资源竞争，根据不同语句的复杂性来控制它的优先级。

因此使用线程池可以将不同的SQL控制在一个合理的连接数范围，使数据库在高并发的场景下保持较高性能。



上图是用库存注释与语句队列后的性能对比，可以看到和原生性能相比翻了40倍不止。

**（六）监控报警**



监控报警是系统调优里必不可少的一部分。

监控有RDS自带的监控，资源层监控，引擎层监控，慢日志的监控，收费的有审计日志，以及自治服务会基于上面RDS自带的以及审计日志做一些分析，在这个基础上做一些报警。此外，分布式监控可以监控不同组件的瓶颈。



**三、流程管理和生态工具**

基于生态工具的保障，可以从变更流程、稳定性和数据恢复这三方面来介绍。

**（一）变更流程**

在大促期间，通过增加审批节点与流程，很好地提升我们的稳定性。



可以通过设置DMS来执行的这些SQL，不让它超过一定的时间。也可以针对某一类的DDL或者是DML不允许执行，设置安全规则。



**（二）稳定性相关**

**1.****设置超时时间（通用）**

稳定性相关，不要因为人为的因素给系统造成压力，可以通过设置DMS来执行的这些SQL，不让它超过一定的时间，超过一定时间则Kill掉。



**2.** **禁用部分语法(企业版)**

可针对DDL、DML设置部分语法不允许在大促期间执行。



**3.** **异步清理大表**

Ø  **内核特性**



AliSQL支持通过异步删除大文件的方式保证系统稳定性。使用InnoDB引擎时，直接删除大文件会导致POSIX文件系统出现严重的稳定性问题，因此InnoDB会启动一个后台线程来异步清理数据文件。当删除单个表空间时，会将对应的数据文件先重命名为临时文件，然后清除线程将异步、缓慢地清理文件。

路径：RDS管理控制台->实例列表->参数设置

 

**4.** **突发SQL访问控制**

Ø  内核特性--并发控制

当业务流量突然暴涨，或出现Bad SQL时，DBA要考虑做限流，止损恢复业务。AliSQL设计了基于语句规则的并发控制，Statement Concurrency Control，简称 CCL。

Ø  DAS限流

为防止数据库压力过大，一般都会在应用端做优化和控制。但在以下场景，也需要在数据库端做优化控制，如：

1）某类SQL并发急剧上升；

2）有数据倾斜SQL；

3）未创建索引SQL。



**（三）数据恢复**

**1.DMS****数据追踪**

在大促高峰期间，经常会人工做一些数据变更，如果数据变更出错的话，可以通过数据追踪恢复，最快五分钟内可完全恢复。



Ø  **使用场景**

1）MySQL 5.5/5.6/5.7/8.0版本；

2）DELETE/UPDATE/INSERT；

3）少量数据。

Ø  **功能**

1）在线搜索日志内容，无需手工下载 Binlog；

2）支持数据的插入/更新/删除日志搜索，无需手工解析Binlog；

3）支持逐条数据恢复，无需手工生成回滚语句。

Ø  **支持的Binlog**

OSS Binlog（RDS会定时将Binlog备份到OSS上）；

本地热Binlog（数据库服务器上Binlog）。

 

**（二）控制台克隆实例/库表级别恢复**

数据恢复还可通过备份来实现，可分为克隆整个实例恢复与库表级别恢复，还可以通过DBS数据库备份来恢复。

路径：RDS管理控制台->实例列表->单击实例ID->备份恢复



总结以上内容，系统调优主要从内容评估、性能评测、架构调优、实例调优以及内核调优等方面实现，可以借助监控报警来发现问题、解决问题，最终实现系统优化。