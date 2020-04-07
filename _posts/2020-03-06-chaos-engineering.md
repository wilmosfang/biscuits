---
layout: post
title: "Overview of Chaos engineering"
date: 2020-03-06 16:27:11
image: '/assets/img/'
description: 'Chaos Engineering 概况'
main-class: tools
color: 
tags: 
 - tools
 - chaos
categories: 
 - tools
twitter_text: 'Overview of Chaos Engineering'
introduction: 'Overview of Chaos Engineering'
---

# 前言

**Chaos Engineering** 最近在国内非常火热，于是我也稍作了了解，这里将学习到的东西汇总出来与大家分享

这次换种形式，通过 Q&A 的方式来进行呈现


部分内容参考了 **[InfoQ][infoq]**

---


# 内容


## 混沌工程发展简介

2008年8月，Netflix 主要数据库的故障导致了三天的停机， DVD租赁业务中断，多个国家的大量用户受此影响

之后 Netflix工程师着手寻找替代架构，并在2011年起，逐步将系统迁移到 AWS上，运行基于微服务的新型分布式架构

这种架构消除了单点故障，但也引入了新的复杂性类型，需要更加可靠和容错的系统。为此， Netflix 工程师创建了 Chaos Monkey，会随机终止在生产环境中运行的 EC2实例。工程师可以快速了解他们正在构建的服务是否健壮，有足够的弹性，可以容忍计划外的故障。

至此，混沌工程开始兴起。

![chaos](/assets/img/chaos/chaos01.png)

上图中展示了混沌工程从2010年演进发展的时间线：

* 2010年 Netflix 内部开发了 AWS 云上随机终止 EC2 实例的混沌实验工具： Chaos Monkey
* 2011年 Netflix 释出了其猴子军团工具集： Simian Army
* 2012年 Netflix 向社区开源由 Java 构建 Simian Army，其中包括 Chaos Monkey V1 版本
* 2014年 Netflix 开始正式公开招聘 Chaos Engineer
* 2014年 Netflix 提出了故障注入测试（FIT），利用微服务架构的特性，控制混沌实验的爆炸半径
* 2015年 Netflix 释出 Chaos Kong ，模拟AWS区域（Region）中断的场景
* 2015年 Netflix 和社区正式提出混沌工程的指导思想 – Principles of Chaos Engineering
* 2016年 Kolton Andrus（前 Netflix 和 Amazon Chaos Engineer ）创立了 Gremlin ，正式将混沌实验工具商用化
* 2017年 Netflix 开源 Chaos Monkey 由 Golang 重构的 V2 版本，必须集成 CD 工具 Spinnaker 来使用
* 2017年 Netflix 释出 ChAP （混沌实验自动平台），可视为应用故障注入测试（FIT）的加强版
* 2017年 由Netflix 前混沌工程师撰写的新书“混沌工程”在网上出版
* 2017年 Russell Miles 创立了 ChaosIQ 公司，并开源了 chaostoolkit 混沌实验框架


## 什么是 Chaos Engineering

**Chaos Engineering** 源自 Netflix 工程师合著的 Chaos Engineering 一书

Chaos Engineering 混沌工程从出现到标准化成为一门学科，是伴随着 Netflix 过去多年时间里同稳定性持续战斗的历程一起成长起来的，这是每一次故障带来的深度思考，抽象而成的理论和实践的结合

哪么什么是 Chaos Engineering 呢

> 在分布式系统上进行由经验指导的受控实验，观察系统行为并发现系统弱点，以建立对系统在规模增大时因意外条件引发混乱的能力和信心

这个里面提到的点有好多，涉及到 **分布式系统、经验、受控实验、系统弱点、规模、意外条件、混乱、能力、信心**

我的理解是

> 在生产环境中，通过受控实验模拟实际环境中可能出现的异常，来发现系统弱点的过程

~~~
个人的看法，混沌工程是传统 QA 的延伸，它们的区别在于， 传统 QA 面对的是一个假定预期状态确定的逻辑， 而混沌工程面对的是一个假定预期状态并不确定的逻辑

所以某种角度上来讲，混沌工程，是一个非常高明的称谓，它更切近事物本身的运行规律，它从一开始就承认和包含了事物运行过程的不确定性

要知道事物的运行规律从微观级别上来讲，是种概率，只是在宏观状态下，呈现出某种看似确定的必然

所以首先承认面对的是一个并不全知的系统，然后再通过一些工程实践来去发掘这些信息，可能是一个更切实的态度
~~~

## 关于假定

传统 QA 面对的是一个假定预期状态确定的逻辑

Peter Deutsch 提出的分布式系统八大谬论，概括了程序员可能对分布式系统做出的错误假设：

* 网络是可靠的
* 延迟是零
* 带宽是无限的
* 网络是安全的
* 拓扑结构不会变
* 存在管理员这样的角色
* 传输成本是零
* 网络是同质的

当以上的假定在现实环境中并不能确保时，基于这些假设而编制的逻辑就可能出现问题

## 混沌工程的实质

这里我借用下面的框架来进行分析

### 质料

混沌工程内涵是一类工程思想与一套方法论的结合体

~~~
它是一套思想一套方法论，存在于人的脑子里
落实为一些工具和流程，作用于系统
~~~

### 形式

混沌工程是通过一系列的工具和流程模拟实际环境中可能出现的异常，来获取更多系统弱点信息的过程

~~~
人们依据这套摸索出来的指导体系，结合一些工具来研究和获取生产系统本身隐含特质的一种实践

如果这种隐含的特质正好是系统的弱点，就是这项活动的意义
~~~



### 动力

主动可控小范围的试错，比被动失控大范围的业务损失更便宜

~~~
比用户更先知道系统的问题总归是好的

想尽办法找出隐藏的风险，以让价值交付过程更为通畅和稳定
~~~

### 目的

进一步加强生产系统的健壮性

~~~
抗异常干扰力或自愈能力或弹性

与之相对应的就是脆性，减少脆性

就像打疫苗可以预防疾病一样，我们可以通过混沌工程来提升系统的免疫能力

我们向系统注入故障（比如延迟、CPU 故障、网络黑洞），找出系统潜在的弱点
~~~

## 混沌工程有哪些作用

混沌工程可以理解成一种主动防御的能力，能够提前找到未知的问题，提高系统韧性

* 由于实验是在最小化爆炸范围内进行的，可以减小业务损失，让重大风险在可控范围提前暴露
* 提升系统弹性，持续验证系统对极端场景的容错能力
* 增强团队信心，验证稳定性措施有效性，量化团队价值

引用自 https://www.jianshu.com/p/4bd4f88e24e4

~~~
混沌工程给客户、业务和技术带来的好处

客户：增强的服务可用性和持久性意味着他们的日常生活不会受到影响。
业务：混沌工程有助于避免公司的利润受到损失，降低维护成本，提升工程师的愉悦感和参与度，改进整个公司的事故处理流程。
技术：混沌工程旨在减少事故和轮班待命的负担，更好地了解系统的故障模式，改进系统设计，更快地检测到事故，减少类似事故的发生
~~~

## 混沌工程需要的前提

1. 当前已经具备一些弹性来应对真实环境中的常见异常
2. 监控系统


~~~
在开始应用混沌工程之前，我们必须确保系统是弹性的，也就是当出现了系统错误我们的整个系统还能正常工作。如果不能确保，我们就需要先考虑提升整个系统的健壮性了，因为混沌工程主要是用来发现系统未知的脆弱一面的，如果我们知道应用混沌工程能导致显而易见的问题，那其实就没必要应用了
~~~

引用自：https://www.jianshu.com/p/4bd4f88e24e4


# 混沌工程的生发渊源

随着市场需求的更迭，软件架构的演进，现在的软件架构已经日趋复杂，在分布式的微服务中，应用逻辑的多层依赖关系已经变得越来越无法简单且直观的被理解，系统的脆弱性变得愈发明显，传统的高可用方案已经无法满足对系统脆弱性的补偿

现存且已知的环境异常和风险可以有针对性进行加固

![chaos](/assets/img/chaos/chaos05.png)

但是未知的风险将无从着手，一旦触发却会让业务蒙受巨大损失让组织陷入被动局面

![chaos](/assets/img/chaos/chaos07.png)

混沌工程，就是在这种情景下产生，用来有效发掘此类风险的工程实践

在生产环境中，通过受控实验模拟实际环境中可能出现的异常，来主动发现系统脆弱性的过程

通过一系列工程实践，让团队从无法识别特定风险到可以识别此类风险的一个过程


(混沌工程暂时不包含如何修复此脆弱性的工作)

明确出系统的脆弱点，然后有针对地进行加固

## 混沌工程的指导原则

![chaos](/assets/img/chaos/chaos02.png)

## 如何实践混沌工程

![chaos](/assets/img/chaos/chaos03.jpeg)

完整的混沌工程实验是一个持续性迭代的闭环体系，大致分为以下几个步骤：

1. 确定初步的实验需求和实验对象；

2. 通过实验可行性评估，确定实验范围；

3. 设计合适的观测指标；

4. 设计合适的实验场景和环境；

5. 选择合适的实验工具和平台框架；

6. 建立实验计划，和实验对象的干系人充分沟通，进而联合执行实验过程；

7. 搜集预先设计好的实验指标；

8. 待实验完成后，清理和恢复实验环境；

9. 对实验结果进行分析；

10. 追踪根源并解决问题；

将以上实验场景自动化，并入流水线，定期执行；之后，便可开始增加新的实验范围，持续迭代和有序改进。


~~~
如何规划第一个混沌测试？
计划

混沌工程中最有用的一个问题是：“哪里可能出问题？”通过这种方式不断质疑我们的服务和运行环境，可以帮助我们发现潜在的弱点，并讨论出预期的结果。与风险管理类似，我们因此可以知道哪些场景更有可能发生，以及需要优先测试哪些场景。大家坐下来，在白板上画出服务、依赖和数据存储组件，围绕“哪里可能出问题”的思路展开讨论。对哪里有质疑，就在这个地方注入一个故障。

假设

在知道哪里可能出问题后，在相应的地方注入故障。那么接下来呢？通过对场景展开讨论，可以对预期的结果有一个假设。比如，这将会给客户、服务或依赖带来哪些影响？

评估

要了解系统在压力下运行时的行为，需要对系统的可用性和持久性进行评估。可以使用与客户相关的度量指标来评估，比如每分钟的订单量。从经验来看，如果这些指标出现异常，必须立即停止测试。接下来要评估故障本身，以便验证之前的假设。被影响到的有可能是延迟、每秒的请求数或系统资源。最后，查看仪表盘和告警，看看有没有出现非预期的副作用。

回退计划

制定回退计划是很有必要的，而且要接受回退计划也可能失效的事实。告诉大家将以怎样的方式进行回退，如果是通过手动执行命令的方式，注意不要让与服务实例连接的 ssh 或控制面板出现问题。

修复

在运行了第一个测试之后，应该能够得到一两个有用的结果。最起码已经验证了系统在出现这些故障时是否具备弹性，或者找到了修复问题的方式。这些都是好的产出结果。一方面，我们对系统的行为更有信心，另一方面，我们找到了一个潜在的问题。
~~~


## 混沌测试的执行顺序

我们建议按照以下的顺序来执行混沌工程试验：

Known Known：注意到并且了解原理

Known Unknown：注意到但不了解原理

Unknown Known：没有注意到但是了解原理

Unknown Unknown：既没有注意到也不了解原理的

~~~
越靠前越趋于传统QA的工作范畴
越靠后越趋于混沌工程的工作范畴，因为靠后更具备探索性，探索性是混沌工程的一大特征
~~~

![chaos](/assets/img/chaos/chaos05.png)


部分内容参考了 **[InfoQ][infoq]**


## 混沌注入的层面

为了知道在意外事件产生时系统的反应，混沌工程中关键的一步就是引入异常，一般而言可以从以下几个层面引入异常

![chaos](/assets/img/chaos/chaos04.png)

简单来讲，就是抽离依赖

![chaos](/assets/img/chaos/chaos06.png)


## 混沌工程的工具

混沌工程的工具：FIT， chaos monkey（破坏节点），Chaos Gorilla（破坏整个az），chaos kong（破坏整个region），chaos lambda，Blockade, ChAP， ChaosBlade


|  工具名称 | 最新版本  | 项目维护状态  |  主要构建语言 | 涉及场景  |特定依赖|链接地址|
|:---|:---|:---|:---|:---|:---|:---|
|Chaos Monkey|	2.0.2	|2016.11之后不在功能开发	|Go	|终止EC2实例	|Spinnaker	|https://github.com/Netflix/chaosmonkey|
|Simian Army	|2.5.3	|retired	|Java|	终止EC2实例；阻断网络流量；卸载磁盘卷；CPU/IO/磁盘空间突发过高；杀进程；路由失败；网络丢包；DynamoDB故障……	|无|	https://github.com/Netflix/SimianArmy和https://queue.acm.org/detail.cfm?id=2499552
|orchestrator|	3.1.1|	alive|	Go|	纯MySQL集群故障场景	|无|	https://github.com/github/orchestrator|
|kube-monkey|	0.3.0|	只发布了一个正式版本|	Go|	杀K8s Pods|	针对K8s集群	|https://github.com/asobti/kube-monkey|
|chaostoolkit	|1.2.0|	alive|	Python	|实验框架，可集成多个IaaS或PaaS平台，可使用多个故障注入工具定制场景，可与多个监控平台合作观测和记录指标信息	|通过插件形式支持多个IaaS、PaaS，包括AWS/Azure/Google/K8s|	https://github.com/chaostoolkit/chaostoolkit|
|PowerfulSeal	|2.2.0|	alive	|Python|	杀K8s、Pods，杀容器，杀虚拟机	|支持OpenStack/AWS/本地机器|	https://github.com/bloomberg/powerfulseal|
|toxiproxy|	2.1.4|	alive|	Go|	网络代理，模拟网络故障|	无|	https://github.com/Shopify/toxiproxy|
|Pumba|	0.6.4|	alive|	Go|	杀停删容器；暂停容器内进程；网络延迟；网络丢包;网络带宽限流	|依赖Docker	|https://github.com/alexei-led/pumba|
|blockade|	0.4.0	|停滞	|Python	|杀容器、网络终端、网络延迟、网络丢包、网络分区	|依赖Docker	|https://github.com/worstcase/blockade|
|chaos-lambda	|0.3.0|	停滞|	Python|	杀自动伸缩组的EC2示例	|依赖于Lambda|	https://github.com/bbc/chaos-lambda|
|namazu	|0.2.1|	停滞|	Go|	文件系统故障;网络故障;Java功能调用故障|	针对类Zookeeper分布式系统|	https://github.com/osrg/namazu|
|chaos-monkey-spring-boot|	2.0.2|	alive|	Java|	应用延迟;异常处理;内存过高|	依赖于Spring Boot框架|	https://github.com/codecentric/chaos-monkey-spring-boot|
|byte-monkey|	1.0.0|	停滞|	Java	|异常处理;应用延迟||	https://github.com/mrwilson/byte-monkey|
|chaosblade	|0.4.0|	alive|	Java|	实验框架，目前支持JVM	||https://github.com/chaosblade-io/chaosblade|
————————————————

原文链接：https://blog.csdn.net/u013256816/java/article/details/103998060


## 哪些公司在实践混沌工程

因为微服务架构的普及，很多大型的技术公司都在采用混沌工程，如 Twilio、Netflix、领英、Facebook、谷歌、微软和亚马逊

混沌工程在银行和金融行业也有所应用

2014 年，澳大利亚国家银行从物理基础设施迁移到 AWS，通过混沌工程显著减少了事故数量


* TOC
{:toc}


---

[infoq]:https://www.infoq.cn/article/chaos-engineering-the-history-principles-and-practice/