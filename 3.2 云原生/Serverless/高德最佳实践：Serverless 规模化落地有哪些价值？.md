- [高德最佳实践：Serverless 规模化落地有哪些价值？](https://blog.51cto.com/13778063/2555576)
- [微众银行案例｜容器化实践在金融行业落地面临的问题和挑战](https://www.cnblogs.com/tencent-cloud-native/p/14260889.html)



随着 Serverless 概念的进一步普及，开发者已经从观望状态进入尝试阶段，更多的落地场景也在不断解锁。“Serverless  只适合小场景吗？”、“只能被事件驱动吗？” 这些早期对 Serverless 的质疑正在逐渐消散，用户正在更多的核心场景中，开始采用  Serverless  技术达到提效、弹性、成本优化等目的。作为地图应用的领导者，高德为带给用户更好的出行体验，不断在新技术领域进行探索，在核心业务规模化落地  Serverless，现已取得显著成效。

2020 年的“十一出行节”期间，高德地图创造了记录 ——截止 2020 年 10 月 1 日 13 时 27 分 27 秒，高德地图当日活跃用户突破 1 亿，比 2019 年 10 月 1 日提前 3 时 41 分达成此记录。

期间，Serverless 作为其中一个核心技术场景，平稳扛住了流量高峰期的考验。值得一提的是，由 Serverless  支撑的业务在流量高峰期的表现十分优秀，每分钟函数调用量接近两百万次。这再次验证了 Serverless 基础技术的价值，进一步拓展了技术场景。

# 业务场景

**自主出行**是高德地图的核心业务，涉及到用户出行相关的功能诉求，承载了高德地图 APP 内最大的用户流量。下图为自主出行核心业务中应用 Node FaaS 的部分场景，从左至右依次为：主图场景页、路线规划页、导航结束页。

![1.jpg](https://ucc.alicdn.com/pic/developer-ecology/5a12e32c0ef34fc4ba2e43f1fc3f59d8.jpg)

随着功能的进一步拓展，高德地图从导航工具升级为出行服务平台和生活信息服务入口，进一步拓展了出行相关的生活信息服务场景，带给用户更全面的用户体验。上图功能为**场景推荐卡片**，旨在根据用户出行意图推荐信息，提升用户出行体验。此功能需具备**快速迭代，样式调整高灵活性**的能力。因此，将卡片样式模版存放于云端，通过服务下发的形式渲染至客户端无疑为最优选择，可以满足业务快速灵活迭代的目的。

经过方案评估判断，此场景类型属于**无状态服务**，基于阿里云 Serverless 成熟的生态，高德最终选择接入 Node FaaS（阿里云函数计算）服务能力，出行前端搭建了场景推荐卡片服务。卡片的 **UI 模版获取、数据请求聚合&逻辑处理、拼接生成 Schema** 的能力均在 FaaS 层得到实现，客户端根据服务下发的 Schema 直接渲染展示，达到更加**轻便灵活的目标**。

![2.png](https://ucc.alicdn.com/pic/developer-ecology/f8e30c6ae3ce4cb6a8e0fe6b9160f95b.png)

那么，Serverless 场景在“十一出行节”峰值场景中的具体表现如何？

> 整体服务成功率均大于 99.99% ，总计 100W+ 次触发/分钟，QPS 2W+，各场景的服务平均响应时间均在 60ms 以下，服务稳定性超出预期。

# 业务价值

从对以上业务场景的支撑中，我们可以看出 Serverless 的表现非常优秀。当然你也会问，传统的应用也能带来同样的体验，那么 Serverless 的差异化价值又是什么呢？

## 1. 简单提效

传统 BFF（Back-end For Front-end）层应用会随着时间推移，以及业务需求的增加， 其 BFF 层逐渐变 “富”，  冗余的代码逐渐变多，最后变成开发者的噩梦——“牵一发而动全身”。随着人员迭代变化，模块的开发者也会变化，BFF  层就会慢慢变成一个无人知晓，无人敢动的模块。

当 BFF 层转换成 SFF (Serverless For Front-end) 层之后，会有什么变化？SFF  的职责会变的单一、零运维、成本更低，这些是 Serverless 本身自带的能力，而这些能力可以帮助前端进一步释放生产潜能。开发者不再需要一个富 BFF 层，而只需一个接口或一个 SFF  就可以实现功能，天然解决了“牵一发而动全身”的问题。如果接口停服或者没有流量，那么所用的实例会自动缩零，也就很容易分辨出是哪一个接口函数，后期就可以删掉此接口的函数，有效提升资源利用率。

高德在 Serverless 应用上非常先进，实现了 FaaS 层与研发体系的完全对接，因此，应用从开发、测试、灰度、上线的全生命周期，到具备流控、弹性、容灾等标准化能力，所用的时间较以前缩短了 40%，大大提高了人效。

![3.png](https://ucc.alicdn.com/pic/developer-ecology/2c1eec0296da4047959ed497575bec27.png)

## 2. 弹性以及成本

通过流量趋势数据，我们可以观察到地图场景流量特点——高峰与低峰的落差十分明显。按照传统应用的资源准备，我们需要根据最高峰的流量进行资源准备，所以到了流量低峰期，多准备的机器会有很多冗余，这就造成了成本的浪费。

针对以上情况，高德使用了阿里云函数计算，可以根据流量变化自动扩缩容。然而，提升扩缩容速度的复杂性较大，一直是大企业的专属，但函数计算可以通过毫秒级别的启动优势，将快上快下的扩缩容能力普及给用户，轻松帮助用户实现了计算资源的弹性利用，并且大大降低了成本。

![4.png](https://ucc.alicdn.com/pic/developer-ecology/d6d4c9c6375e41ad9a606156e3ae2b93.png)
![5.png](https://ucc.alicdn.com/pic/developer-ecology/ba98ce1197c745b98dc54da902d2c7ae.png)

## 3. 可观测性

可观测性是应用上线诊断平台的必备属性，要让用户观察到 RT  变化、资源的使用率、系统应用的全链路调用，从而快速诊断出系统应用的瓶颈问题。阿里云函数计算率先与日志服务、云监控、tracing  平台以及函数工作流编排做了完美的融合，用户只需要配置一次，就可以完完整整的享受到以上这些功能，大大降低了用户的学习成本，实现了对应用程序的快速诊断。

![6.png](https://ucc.alicdn.com/pic/developer-ecology/1f1efc3813394d5d98b98b952360e0d3.png)
![7.png](https://ucc.alicdn.com/pic/developer-ecology/db7c26d80cdd44648af12147b6e83244.png)

Serverless 规模化落地的序幕已经拉开， 更多场景正在各行各业中解锁。Serverless  在高德的规模化落地，对于业务方来而言，业务迭代更快更灵活了，为业务创新创造了前提条件；对于前端开发者而言，进一步激活了开发者的生产潜能，提升了极大的能力自信。高德出行业务从 2020  年初的能力试点到“十一出行节”的自主出行核心场景，期间接入了阿里云函数计算，积累了非常宝贵的云原生落地经验，为未来业务整体上云打下了良好基础。