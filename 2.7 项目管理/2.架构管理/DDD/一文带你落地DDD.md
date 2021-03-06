- [一文带你落地DDD](https://juejin.cn/post/7004002483601145863)

# 一.前言

hello，everyone，好久不见。最近几周部门有个大版本发布，一直没有抽出时间来写博。由于版本不断迭代，功能越做越复杂，系统的维护与功能迭代越来越困难。前段领导找我说，能不能在架构上动手做做文章，将架构迁移到DDD。哈哈哈哈，当时我听到这个话的时候瞬间来了精神。说实话，从去年开始从大厂的一些朋友那里接触到DDD，自己平时也会时不时的阅读相关的文章与开源项目，但是一直没有机会在实际的工作中实施。正好借着这次机会可以开始实践一下。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87e68e5d04474526974f35d4363c8cdb~tplv-k3u1fbpfcp-watermark.awebp)

本文由于本文的重点为MVC三层架构如何迁移DDD，因此将先对DDD做一个简要的概念介绍`(细化的领域概念不做过多展开)`,然后对于MVC三层架构迁移至DDD作出迁移方案建议。如有不对之处，欢迎指出，共同进步。

本文尤其感谢一下[lilpilot](https://juejin.cn/user/1978776663097704)在DDD落地方案上给出的宝贵建议。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04d43a3341f94496824c115b888e9b89~tplv-k3u1fbpfcp-watermark.awebp)

**DDD系列博客**

1. [一文带你落地DDD](https://juejin.cn/post/7004002483601145863)
2. [DDD落地之事件驱动模型](https://juejin.cn/post/7005175434555949092)
3. [DDD落地之仓储](https://juejin.cn/post/7006595886646034463)
4. [DDD落地之架构分层](https://juejin.cn/post/7007382308667785253)

> 我的第一本掘金小册[《深入浅出DDD》](https://juejin.cn/book/7049273428938850307)已经在掘金上线，欢迎大家试读~

**DDD的微信群我也已经建好了，由于文章内不能放二维码，大家可以加我微信`baiyan_lou`,备注DDD交流，我拉你进群，欢迎交流共同进步。**

# 二.DDD是什么

## 2.1.DDD简介

相信了解过DDD的同学都听过网上那种官方的介绍：

- Domain Drive Design（领域驱动设计）

- 六边形架构模型

- 领域专家、设计人员、开发人员都能理解的通用语言作为相互交流的工具

- ....

  说的都多多少少抽象点了，听君一席话，如听一席话，哈哈哈

在我看来常规在MVC三层架构中，我们进行功能开发的之前，拿到需求，解读需求。往往最先做的一步就是先设计表结构，在逐层设计上层dao，service，controller。对于产品或者用户的需求都做了一层自我理解的转化。

**众所周知，人才是系统最大的bug。**

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/526ecf6adac34d5d8afd545c8d7f05f8~tplv-k3u1fbpfcp-watermark.awebp)

![image-20210904135645004.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7899df5b15aa471095382d0dc94d5186~tplv-k3u1fbpfcp-watermark.awebp)

用户需求在被提出之后经过这么多层的转化后，特别是研发需求在数据库结构这一层转化后，将业务以主观臆断行为进行了转化。一旦业务边界划分模糊，考虑不全。大量的逻辑补充堆积到了代码层实现，变得越来越难维护，到处是if/else，传说中***一样代码。

![image-20210904140321557.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b355a6e3adf4f6a8cb13571980655f0~tplv-k3u1fbpfcp-watermark.awebp)

**DDD所要做的就是**

- 消除信息不对称
- 常规MVC三层架构中自底向上的设计方式做一个反转，以业务为主导，自顶向下的进行业务领域划分
- 将大的业务需求进行拆分，分而治之

> 说到这里大家可能还是有点模糊DDD与常见的mvc架构的区别。这里以电商订单场景为例。假如我们现在要做一个电商订单下单的需求。涉及到用户选定商品，下订单，支付订单，对用户下单时的订单发货。
>
> MVC架构里面，我们常见的做法是在分析好业务需求之后，就开始设计表结构了，订单表，支付表，商品表等等。然后编写业务逻辑。这是第一个版本的需求，功能迭代饿了，订单支付后我可以取消，下单的商品我们退换货，是不是又需要进行加表，紧跟着对于的实现逻辑也进行修改。功能不断迭代，代码就不断的层层往上叠。
>
> DDD架构里面，我们先进行划分业务边界。这里面核心是订单。那么订单就是这个业务领域里面的聚合逻辑体现。支付，商品信息，地址等等都是围绕着订单而且。订单本身的属性决定之后，类似于地址只是一个属性的体现。当你将订单的领域模型构建好之后，后续的逻辑边界与仓储设计也就随之而来了。

## 2.2.为什么要用DDD

- 面向对象设计，数据行为绑定，告别贫血模型
- 降低复杂度，分而治之
- 优先考虑领域模型，而不是切割数据和行为
- 准确传达业务规则，业务优先
- 代码即设计
- 它通过边界划分将复杂业务领域简单化，帮我们设计出清晰的领域和应用边界，可以很容易地实现业务和技术统一的架构演进
- 领域知识共享，提升协助效率
- 增加可维护性和可读性，延长软件生命周期
- 中台化的基石

## 2.3.DDD术语介绍

战略设计：限界上下文、通用语言，子域

战术设计：聚合、实体、值对象、资源库、领域服务、领域事件、模块

![1595145053316-e3f10592-4b88-479e-b9b7-5f1ba43cadcb.jpeg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b811a0142f344a18869f3fb515fc715e~tplv-k3u1fbpfcp-watermark.awebp)

### 2.3.1.限界上下文与通用语言

限界上下文是一个显式的语义和语境上的边界，领域模型便存在于边界之内。边界内，通用语言中的所有术语和词组都有特定的含义。

通用语言就是能够简单、清晰、准确描述业务涵义和规则的语言。

把限界上下文拆解开看。限界就是领域的边界，而上下文则是语义环境。  通过领域的限界上下文，我们就可以在统一的领域边界内用统一的语言进行交流。

```
域是问题空间，限界上下文是解决空间
```

### 2.3.2.上下文组织和集成模式

防腐层(Anticorruption Layer)：简称ACL，在集成两个上下文，如果两边都状态良好，可以引入防腐层来作为两边的翻译，并且可以隔离两边的领域模型。

![image-20210904143337032.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d135f788ba844ce9bc262d30f1276dc~tplv-k3u1fbpfcp-watermark.awebp)

### 2.3.3.实体

DDD中要求实体是唯一的且可持续变化的。意思是说在实体的生命周期内，无论其如何变化，其仍旧是同一个实体。唯一性由唯一的身份标识来决定的。可变性也正反映了实体本身的状态和行为。

```
实体 = 唯一身份标识 + 可变性【状态 + 行为】
```

### 2.3.4.值对象

当你只关心某个对象的属性时，该对象便可作为一个值对象。 我们需要将值对象看成不变对象，不要给它任何身份标识，还应该尽量避免像实体对象一样的复杂性。

```
值对象=将一个值用对象的方式进行表述，来表达一个具体的固定不变的概念。
```

### 2.3.5.聚合

聚合是领域对象的显式分组，旨在支持领域模型的行为和不变性，同时充当一致性和事务性边界。

```
我们把一些关联性极强、生命周期一致的实体、值对象放到一个聚合里。
```

### 2.3.6.聚合根

聚合的根实体，最具代表性的实体

### 2.3.7.领域服务

当一些逻辑不属于某个实体时，可以把这些逻辑单独拿出来放到领域服务中 `理想的情况是没有领域服务`，如果领域服务使用不恰当慢慢又演化回了以前逻辑都在service层的局面。

可以使用领域服务的情况：

- 执行一个显著的业务操作
- 对领域对象进行转换
- 以多个领域对象作为输入参数进行计算，结果产生一个值对象

### 2.3.8.应用服务

应用服务是用来`表达用例和用户故事`的主要手段。

**应用层通过应用服务接口来暴露系统的全部功能。** 在应用服务的实现中，它负责编排和转发，它将要实现的功能委托给一个或多个领域对象来实现，它本身只负责处理业务用例的执行顺序以及结果的拼装。通过这样一种方式，它隐藏了领域层的复杂性及其内部实现机制。

应用层相对来说是较“薄”的一层，除了定义应用服务之外，在该层我们可以进行安全认证，权限校验，持久化事务控制，或者向其他系统发生基于事件的消息通知，另外还可以用于创建邮件以发送给客户等。

`应用层作为展现层与领域层的桥梁`。展现层使用VO（视图模型）进行界面展示，与应用层通过DTO（数据传输对象）进行数据交互，从而达到展现层与DO（领域对象）解耦的目的。

### 2.3.9.工厂

职责是创建完整的聚合

- 工厂方法
- 工厂类

**领域模型中的工厂**

- 将创建复杂对象和聚合的职责分配给一个单独的对象，它并不承担领域模型中的职责，但是领域设计的一部份
- 对于聚合来说，我们应该一次性的创建整个聚合，并且确保它的不变条件得到满足
- 工厂只承担创建模型的工作，不具有其它领域行为
- 一个含有工厂方法的聚合根的主要职责是完成它的聚合行为
- 在聚合上使用工厂方法能更好的表达通用语言，这是使用构造函数所不能表达的

**聚合根中的工厂方法**

- 聚合根中的工厂方法表现出了领域概念
- 工厂方法可以提供守卫措施

**领域服务中的工厂**

- 在集成限界上下文时，领域服务作为工厂
- 领域服务的接口放在领域模型内，实现放在基础设施层

### 2.3.10.资源库【仓储】

是聚合的管理，仓储介于领域模型和数据模型之间，主要用于聚合的持久化和检索。它隔离了领域模型和数据模型，以便我们关注于领域模型而不需要考虑如何进行持久化。

我们将暂时不使用的领域对象从内存中持久化存储到磁盘中。当日后需要再次使用这个领域对象时，根据 key 值到数据库查找到这条记录，然后将其恢复成领域对象，应用程序就可以继续使用它了，这就是**领域对象持久化存储的设计思想**

### 2.3.11.事件模型

领域事件是一个领域模型中极其重要的部分，用来表示领域中发生的事件。忽略不相关的领域活动，同时明确领域专家要跟踪或希望被通知的事情，或与其他模型对象中的状态更改相关联

**领域事件 = 事件发布 + 事件存储 + 事件分发 + 事件处理。**

> 比如下订单后，给用户增长积分与赠送优惠券的需求。如果使用瀑布流的方式写代码。一个个逻辑调用，那么不同用户，赠送的东西不同，逻辑就会变得又臭又长。这里的比较好的方式是，用户下订单成功后，发布领域事件，积分聚合与优惠券聚合监听订单发布的领域事件进行处理。

## 2.4.DDD架构总览

### 2.4.1.架构图

严格分层架构:某层只能与直接位于的下层发生耦合。

松散分层架构:允许上层与任意下层发生耦合

**依赖倒置原则**

高层模块不应该依赖于底层模块，两者都应该依赖于抽象

抽象不应该依赖于实现细节，实现细节应该依赖于接口

简单的说就是面向接口编程。

按照DIP的原则，领域层就可以不再依赖于基础设施层，基础设施层通过注入持久化的实现就完成了对领域层的解耦，采用依赖注入原则的新分层架构模型就变成如下所示：

![image-20210904145125083.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef5dacc05e814e27ad17d556e84503a8~tplv-k3u1fbpfcp-watermark.awebp)

从上往下

**第一层**为用户交互层，web请求，rpc请求，mq消息等外部输入均被视为外部输入的请求，可能修改到内部的业务数据。

**第二层**为业务应用层，与MVC中的service不同的不是，service中存储着大量业务逻辑。但在应用服务的实现中（以功能点为维度），它负责编排、转发、校验等。

**第三层**为领域层，聚合根是里面最高话事人。核心逻辑均在聚合根中体现【充血模型】，如果当前聚合根不能处理当前逻辑，需要其他聚合根的配合时，则在聚合根的外部包一层领域服务去实现逻辑。当然，理想的情况是不存在领域服务的。

**第四层**为基础设施层，为其他层提供技术实现支持

相信这里大家还看见了应用服务层直接调用仓储层的一条线，这条线是什么意思呢？

领域模型的建立是为了控制对于数据的增删改的业务边界，至于数据查询，不同的报表，不同的页面需要展示的数据聚合不具备强业务领域，因此常见的会使用CQRS方式进行查询逻辑的处理。

### 2.4.2.六边形架构(端口与适配器)

对于每一种外界类型，都有一个适配器与之对应。外界接口通过应用层api与内部进行交互。

对于右侧的端口与适配器，我们可以把资源库看成持久化的适配器。

![image-20210904150651866.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84476e5eff804a769a4b1ee88c6d8857~tplv-k3u1fbpfcp-watermark.awebp)

### 2.4.3.命令和查询职责分离--CQRS

- 一个对象的一个方法修改了对象的状态，该方法便是一个命令(Command)，它不应该返回数据，声明为void。
- 一个对象的一个方法如果返回了数据，该方法便是一个查询(Query)，不应该通过直接或者间接的手段修改对象状态。
- 聚合只有Command方法，没有Query方法。
- 资源库只有add/save/fromId方法。
- 领域模型一分为二，命令模型(写模型)和查询模型(读模型)。
- 客户端和查询处理器 客户端：web浏览器、桌面应用等 查询处理器：一个只知道如何向数据库执行基本查询的简单组件，查询处理器不复杂，可以返回DTO或其它序列化的结果集，根据系统状态自定
- 查询模型：一种非规范化的数据模型，并不反映领域行为，只用于数据显示
- 客户端和命令处理器 聚合就是命令模型 命令模型拥有设计良好的契约和行为，将命令匹配到相应的契约是很直接的事情
- 事件订阅器更新查询模型
- 处理具有最终一致性的查询模型

### 2.4.4.事件驱动架构

落地指导与实践：[DDD落地之事件驱动模型](https://juejin.cn/post/7005175434555949092)

- 事件驱动架构可以融入六边型架构，融合的比较好，也可以融入传统分层架构
- 管道和过滤器
- 长时处理过程
  1. 主动拉取状态检查：定时器和完成事件之间存在竞态条件可能造成失败
  2. 被动检查，收到事件后检查状态记录是否超时。问题：如果因为某种原因，一直收不到事件就一直不过期
- 事件源
  1. 对于聚合的每次命令操作，都至少一个领域事 件发布出去，表示操作的执行结果
  2. 每一个领域事件都将被保存到事件存储中
  3. 从资源库获取聚合时，将根据发生在聚合上的 事件来重建聚合，事件的重放顺序与其产生顺序相同
  4. 聚合快照：将聚合的某一事件发生时的状态快 照序列化存储下来。以减少重放事件时的耗时

# 三.落地分享

## 3.1.事件风暴

EventStorming则是一套Workshop（可以理解成一个类似于头脑风暴的工作坊）方法。DDD出现要比EventStorming早了10多年，而EventStorming的设计虽然参考了DDD的部分内容，但是并不是只为了DDD而设计的，是一套独立的通过协作基于事件还原系统全貌，从而快速分析复杂业务领域，完成领域建模的方法。

![image-20210904152542121.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/960e88e7ec534d129af029f01d37d3b3~tplv-k3u1fbpfcp-watermark.awebp)

针对老系统内的业务逻辑，根据以上方式进行业务逻辑聚合的划分

例如电商场景下购车流程进行事件风暴

![image-20210904152737731.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b104e9d6bcd04ae69dad0a440d85de37~tplv-k3u1fbpfcp-watermark.awebp)

## 3.2.场景识别

事件风暴结束明确业务聚合后，进行场景识别与层级划分

![image-20210904153035722.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc789235fc5e4b8fb584585a737ffd94~tplv-k3u1fbpfcp-watermark.awebp)

## 3.3.包模块划分

![图片2.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02d8ef7af85e4e76b76189a1839d5273~tplv-k3u1fbpfcp-watermark.awebp)

## 3.4.迁移说明

### 3.4.1.仓储层

在我们日常的代码中，使用Repository模式是一个很简单，但是又能得到很多收益的事情。最大的收益就是可以彻底和底层实现解耦，让上层业务可以快速自发展。

以目前逆向模型举例，现有

- OrderDO
- OrderDAO

可以通过以下几个步骤逐渐的实现Repository模式：

1. 生成Order实体类，初期字段可以和OrderDO保持一致
2. 生成OrderDataConverter，通过MapStruct基本上2行代码就能完成
3. 写单元测试，确保Order和OrderDO之间的转化100%正确
4. 生成OrderRepository接口和实现，通过单测确保OrderRepository的正确性
5. 将原有代码里使用了OrderDO的地方改为Order
6. 将原有代码里使用了OrderDAO的地方都改为用OrderRepository
7. 通过单测确保业务逻辑的一致性。

有一点要注意，目前我们用mybatis，dao操作都是含有业务含义的，正常的repository不应该有这种方法，目前repository中的含有业务含义的方法只是兼容方案，最终态都要干掉的。

极端DDD推崇者要求在repository中只存在save与byId两个聚合方法。这个当然需要根据实际业务场景来决定，但是还是建议仅保存这两个方法，其他业务需求查询聚合的方法单独开一个queryRepository实现不同数据的查询聚合与页面数据展示。保证数据的`增删改入口唯一`

### 3.4.2. 隔离三方依赖-adapter

思想和repository是一致的，以调用payApi为例：

1. 在domain新建adapter包
2. 新建PayAdapter接口
3. 在infrastructure中定义adapter的实现，转换内部模型和外部模型，调用pay接口，返回内部模型dto
4. 将原先业务中调用rpc的地方改成adapter
5. 单测对比rpc和adapter，保证正确性

### 3.4.3. 抽离技术组件

同样是符合六边形架构的思想，把mqProducer，JsonUtil等技术组件，在domain定义接口，在infrastructure写实现，替换步骤和adapter类似。

### 3.4.4. 业务流程模块化-application

如果是用能力链的项目，能力链的service就可以是application。如果原先service中的业务逻辑混杂，甚至连参数组装都是在service中体现的。那么需要把逻辑归到聚合根中，当前聚合根无法完全包裹的，防止在领域模型中体现。在应用服务层中为能力链的体现。

### 3.4.5. CQRS参数显式化

能力链项目，定义command，query包，通过能力链来体现Command，Query，包括继承CommandService、QueryService，Po继承CommandPo，QueryPo

非能力链项目，在application定义command，query包，参数和类名要体现CQRS。

### 3.4.6. 战略设计-domain

重新设计聚合和实体，可能和现有模型有差异，如果模型差距不大，直接将能力点内的逻辑，迁移到实体中，

将原来调用repository的含业务含义的方法，换成save，同时删除含业务含义的方法，这个时候可以考虑用jpa替换mybatis，这里就看各个子域的选择了，如果用jpa的话 dao层可以干掉。至此，原biz里的大多数类已迁移完成。

# 四.迁移过程中可能存在的疑问

迁移过程中一定会存在或多或少不清楚的地方，这里我分享一下我在迁移的过程中遇到的问题。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14ac875ee4c746839adf3b64472b75ca~tplv-k3u1fbpfcp-watermark.awebp)

```
1.领域服务与应用服务的实际应用场景区别
```

应用服务：可以理解为是各种方法的编排，不会处理任务业务逻辑，比如订单数修改，导致价格变动，这个逻辑体现在聚合根中，应用服务只负责调用。

领域服务：聚合根本身无法完全处理这个逻辑，例如支付这个步骤，订单聚合不可能支付，所以在订单聚合上架一层领域服务，在领域服务中实现支付逻辑。应用服务调用领域服务。

```
2.聚合根定义的业务边界是什么？
```

不以表结构数据进行业务逻辑的划分，一个业务体为一块业务。比如一个订单涉及商品，收货地址，发货地址，个人信息等等。以实体与值对象的方式在聚合内进行定义。

```
3.一个command修改一个聚合时，会关联修改到别的关联表，这个关联表算不算聚合
```

关联表不算聚合，算值对象

```
4.应用服务层如果调用rpc是否必须使用adapter
```

是的，必须使用，屏蔽外部依赖对于当前业务逻辑的影响。设想一下，你现在需要调用rpc接口，返回的字段有100，你要取其中50个字段。隔了一段时间，调用方改了接口逻辑的返回，数据被包含在实体内。而你调用这个接口的地方特别多，改动就很大了。但是如果有了适配器这一层，你只要定义本身业务需要的数据结构，剩下的业务不需要考虑，完全新人适配器可以将你想要的数据从rpc中加载到。

```
5.聚合根内部逻辑无法单独处理时，放到领域服务内的话，是否可以调用其他聚合根的领域服务或者应用服务，加入业务强绑定形式，聚合根内部如果需要调用service服务或者仓储时如何做。
```

可以这么做，但是逻辑要保证尽量内聚。

```
6.事件通知模式，比如是强绑定形式的，是否还是此种方式，还是与本聚合根无关的逻辑均走事件通知
```

强依赖形式的走逻辑编排，比如订单依赖支付结果进行聚合修改则走应用服务编排。订单支付后发送优惠券，积分等弱耦合方式走事件通知模式。

```
7.聚合根，PO,DTO,VO的限界
```

po是数据库表结构的一一对应。

dto是数据载体，贫血模型，仅对数据进行装载。

vo为dto结构不符合前端展示要求时的包装。

聚合根为一个或者多个po的聚合数据，当然不仅仅是po的组合，还有可能是值对象数据，充血模型，内聚核心业务逻辑处理。

```
8.查询逻辑单独开设一个repository，还是可以在聚合根的仓储中，划分的依据是什么
```

单独开设一个仓储。聚合根的仓储应该查询结果与save的参数均为聚合根，但是业务查询可能多样，展示给前端的数据也不一定都是聚合根的字段组成，并且查询不会对数据库造成不可逆的后果，因此单独开设查询逻辑处理，走CQRS模式。

```
9.返回的结果数据为多个接口组成，是否在应用服务层直接组合
```

不可以，需要定义一个assember类，单独对外部依赖的各种数据进行处理。

```
10.save方法做完delete，insert，update所有方法吗？
```

delete方法单独处理，可以增加一个delete方法，insert与update方法理论上是需要保持统一方法的。

```
11.查询逻辑如果涉及到修改聚合根怎么处理
```

简单查询逻辑直接走仓储，复杂逻辑走应用服务，在应用服务中进行聚合根数据修改。

```
12.逻辑处理的service放置在何处
```

如果为此种逻辑仅为某个聚合使用，则放置在对应的领域服务中，如果逻辑处理会被多个聚合使用，则将其单独定义一个service，作为一个工具类。

# 五.总结

本文对DDD做了一个不算深入的概念，架构的介绍。后对现在仍旧还是被最多使用的MVC三层架构迁移至DDD方案做了一个介绍，最后对可能碰到的一些细节疑问点做了问答。

**当然不是所有的业务服务都合适做DDD架构，DDD合适产品化，可持续迭代，业务逻辑足够复杂的业务系统，中小规模的系统与团队还是不建议使用的，毕竟相比较与MVC架构，成本很大。**

demo演示：[DDD-demo](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Flouyanfeng25%2Fddd-demo)

关于MVC分层的微服务架构博主在之前的文章中也给出过一些设计规范，感兴趣的大家可以去看看：

1.[看完这篇，你就是架构师](https://juejin.cn/post/6968343477352398879)

2.[求求你，别写祖传代码了](https://juejin.cn/post/6968019789675495454)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8d25328d5534e4ba3d518a21c9feee3~tplv-k3u1fbpfcp-watermark.awebp)

# 六.更多DDD学习资料

**博客资料:**

[ThoughtWork DDD系列](https://link.juejin.cn?target=https%3A%2F%2Finsights.thoughtworks.cn%2Ftag%2Fdomain-driven-design%2F)

[张逸 DDD系列](https://link.juejin.cn?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Fuser%2F1327231)

[欧创新 DDD系列](https://link.juejin.cn?target=https%3A%2F%2Fwww.infoq.cn%2Farticle%2Fs_LFUlU6ZQODd030RbH9)

**代码示例：**

[阿里COLA](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Falibaba%2FCOLA)

> [github.com/citerus/ddd…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fciterus%2Fdddsample-core)

> [github.com/YaoLin1/ddd…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FYaoLin1%2Fddddemo)

> [github.com/ddd-by-exam…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fddd-by-examples%2Ffactory)

> [github.com/Sayi/ddd-ca…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FSayi%2Fddd-cargo)