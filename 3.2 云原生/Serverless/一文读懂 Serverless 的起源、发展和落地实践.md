- [一文读懂 Serverless 的起源、发展和落地实践](https://mp.weixin.qq.com/s/Zh-hBFrDijYNsc46t4XmTA)

## 1 Serverless 的发展轨迹

2012 年，Serverless 这个单词第一次出现，由 Iron 公司提出，字面意思就是不需要服务器。但是真正被大家所熟知，是在 2014 年  AWS 推出 Lambda 的时候。Lambda 产品的推出开启了云计算的新时代，之后所有的大厂都在跟进，比如微软、谷歌、IBM  都先后推出自己的 Serverless 产品。

国内是在 2017 年的时候，阿里云和腾讯云先后推出了自己的 Serverless 平台。但这个时候，都是指 FaaS（Function as a  Service）。接着到 2018 年，大家开始渐渐接触到 Serverless，更多还是支付宝小程序和微信小程序的云开发平台。随后到 2019 年，国内其他厂商如百度、华为、字节也都开始做 Serverless，现在 Serverless 已经成了各大云厂商的标配。

![image-20220117230744290](https://gitee.com/er-huomeng/img/raw/master/img/image-20220117230744290.png)

## 2 Serverless 是云计算的 2.0

为什么大家都要做 Serverless 呢？因为大家都认为 Serverless 是云计算 2.0。随着云计算的发展，Serverless 已经成为一个技术趋势、一个理念、一个云的发展方向。

云计算领域有两篇非常著名的论文，是伯克利大学分别在 2009 年和 2019 年发表的。伯克利大学在 2009 年发表的一篇关于云计算的论文，预测了云计算的发展，比如计算资源可以按需索取、支持弹性、简化运维等，这些预测目前都已经实现。

而伯克利在 2019 年 2 月发表的第二篇论文中，预测 Serverless 是云计算下一个十年的发展方向。论文里也给出了关于 Serverless  的定义，简单讲就是Serverless Computing，由 FaaS + BaaS（Backend as a Service）构成一个  Serverless 软件架构。特点就是能够按需弹性、按需付费，这与 CNCF  的定义是相似的，应用以微服务或者函数的形式，拆解并部署到云上，能够按需去做弹性伸缩，按需付费，不用关心底层资源。

![image-20220117230802274](https://gitee.com/er-huomeng/img/raw/master/img/image-20220117230802274.png)

### 2.1 Serverless 是云原生发展的高级阶段

Serverless 跟云原生有什么关系呢？Serverless 的出现，就像人类的演进过程，代表着生产力的解放，极大提升了客户用云的效率。Serverless 在其之上封装了容器技术，是云原生的高级阶段。

Serverless 是对用户强调 No Server，本质并不是不需要服务器，而是将服务器全权托管给了云厂商，用户不用去关心，不用去管理，只用把业务部署到平台上来，只需聚焦业务逻辑代码，能够根据实际请求进行弹性伸缩，不用再去关心资源够不够。

![image-20220117230821421](https://gitee.com/er-huomeng/img/raw/master/img/image-20220117230821421.png)

## 3 Serverless 的核心价值

从物理机到 Serverless，就像我们买车一样，如果要买一辆私家车，这个车的车况保险全部要自己关心，然后你要自己开；到了虚拟机之后，我们把业务 host 到云上，就像汽车租赁；然后再到网约车，我不用买车，不用关心车况，我们要从 A 点到 B 点，只需要打个车，完全按需付费，按需弹性。

抽象出来其实就是有 3 个核心价值：

- 第一个是**弹性伸缩**，它比较省事。比如说我们刚才有电商场景，需要弹性、扛大流量，Serverless 能够及时把资源弹出来。
- 第二个特点，**按需付费**，我们用多少资源就花多少钱，不用为闲置资源来买单。
- 第三个就是**简化运维**，能够帮用户省去资源管理的烦恼。

![image-20220117230842168](https://gitee.com/er-huomeng/img/raw/master/img/image-20220117230842168.png)

我们可以更直观来看 Serverless  的价值：首先最上层是业务逻辑，其次是对接的数据库、存储、微服务框架等，往下要建立监控系统、日志系统，以及容灾和高可用等，再到底层还要维护各种各样的 IaaS 资源，如虚拟机集群。而 Serverless  帮用户省掉的是资源层和可观测层，平台负责底层弹性资源，包括所有的日志监控等，用户就只需要关心业务逻辑。

![image-20220117230851042](https://gitee.com/er-huomeng/img/raw/master/img/image-20220117230851042.png)

## 4 Serverless 的软件架构

作为开发者，我们可以直接把镜像或者代码包部署到 Serverless 计算平台上来，省去了整个资源的购买和环境部署这个过程。部署上来之后呢，后端可以跟存储、数据库进行交互，构成完整的  Serverless 架构。之后通过像 LB 或者 HTT  的方式，直接去访问到业务代码，平台会根据用户的请求去做调度和弹性伸缩。Serverless  平台支持负载均衡，应对各种突发流量，用户不用去关心后台资源。

![image-20220117230904987](https://gitee.com/er-huomeng/img/raw/master/img/image-20220117230904987.png)

组件架构有点复杂，本次不展开来讲。对于开发者，需要关注的是绿色部分，即业务代码和服务框架等，以及用什么样的工具和后端 BaaS。Serverless 平台会纳管所有基础设施，会做好消息缓存、流量调度、容灾、高可用等。

![image-20220117230915098](https://gitee.com/er-huomeng/img/raw/master/img/image-20220117230915098.png)

另一个非常重要的组件架构是 Serverless 应用引擎，它的本质是把 K8S 做了封装。如果企业有微服务业务，并且需要部署到 K8S  集群，而维护挑战比较大的话，就可以用这种形态。把开发好的微服务，或者单体应用直接托管到这个平台上来，就能够享受 Serverless  所带来的弹性伸缩和按需付费的价值。

![image-20220117230923870](https://gitee.com/er-huomeng/img/raw/master/img/image-20220117230923870.png)

## 5 Serverless 的落地实践

Serverless 已有多个落地场景，在各个行业，无论是后台服务，还是 REST API 都可以部署到 Serverless 平台上。尤其是 Serverless 音视频处理、轻量 ETL（低门槛数据分析/处理）、事件驱动、任务跑批、应用托管、微服务容器化等场景。

在 Serverless 平台上有非常多的应用 case，比如，如果想要做微服务或者容器化转型，期望降低运维复杂度的同时，也能具备弹性伸缩、便捷发布的能力，就可以直接把服务部署到 Serverless 应用引擎。

![image-20220117230937998](https://gitee.com/er-huomeng/img/raw/master/img/image-20220117230937998.png)

再分享一个案例，想必很多人都有看过欧洲杯，国内是爱奇艺体育在做这个赛事直播，其背后的业务就是部署在 Serverless 应用引擎平台上。

对于爱奇艺体育团队来讲，最大的痛点之一是资源的弹性。因为体育赛事的直播流量有非常大的不确定性，面对流量激增，需要及时能够对后台服务进行扩容，如果按照峰值对资源进行保有，又会造成流量预估不准确的风险，以及一定程度上的资源浪费。

所以，Serverless 应用引擎非常好地匹配了客户痛点问题，不仅解决了弹性扩缩的问题，也提升了资源利用率，同时配套的应用监控，也极大程度上提升了定位问题的效率。

> 推荐阅读：[爱奇艺体育：体验 Serverless 极致扩缩容，资源利用率提升 40%](http://mp.weixin.qq.com/s?__biz=MzI4NzI5MDM1MQ==&mid=2247492523&idx=1&sn=f9eb1b37043cf1b968e3adb3b40ea2d6&chksm=ebcd452bdcbacc3dbd81835e353b6689d9e798aeabda9b4e4ec8a39029b6657f3bd47f5f9ef8&scene=21#wechat_redirect)

![image-20220117230951577](https://gitee.com/er-huomeng/img/raw/master/img/image-20220117230951577.png)

## 6 Serverless 的未来畅想

- 大面积取代 Serverful，变为默认的计算范式：虽然 Serverful 不会完全消失，但随着 Serverless 存在的不足被逐个攻克，Serverlsss 在云计算中所占的比重将会逐渐提升，变成云时代默认的计算范式。
- 拥抱整个容器生态：未来，Serverless 会更多的去拥抱整个容器生态，当下容器是整个业界的一个主流的趋势，Serverless 会和容器做更多的集成，比如镜像部署、镜像加速、以及集成 K8S 很多的能力。
- 加速运维关系的变化：Serverless 将会加速运维关系的转变，运维同学会从资源运维，逐步走向业务运维。
- 复杂任务编排、工具链及可观测等能力提升：Serverless 会加强复杂任务编排、工具链和可观测等方面的能力。因为 Serverless 平台对底层资源做了高度封装，所以一定要把很多的监控指标去透露给用户，通过这些指标来做业务级的管理和管控。

![image-20220117231011204](https://gitee.com/er-huomeng/img/raw/master/img/image-20220117231011204.png)

我们希望 Serverless 能够真正给大家减负，让业务开发和维护变的更加简单，给业务带来更大的价值。