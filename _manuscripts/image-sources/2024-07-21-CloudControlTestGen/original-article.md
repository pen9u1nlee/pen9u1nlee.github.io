---
title: 智能网联汽车云控多交通参与者测评系统
tags: 云控制系统
pageview: false
modify_date: 2024-07-21
aside:
  toc: true
---

> 这是同济大学做的一个封闭测试平台以及实际测试场景，用于高效产生智能网联汽车的长尾场景（例如车祸、鬼探头）及数据。

<!--more-->

![alt text](/img/2024-07-21-CloudControlTestGen/image.png)

上图展示了智能网联汽车出货量和安全性问题。这里我们关注安全性问题：

特斯拉事故：基于纯视觉方案，把侧翻大巴的白色车顶当成了天空，于是直接开创。

萝卜上路的问题：在十字路口，萝卜车判断当前场景需要停车或者慢行，交警没法有效疏导（低情商：撵），于是导致交通拥堵。

测试场的问题：测试场内设置的假人场景和车辆智能的参数能在多大程度上还原真实场景？

![alt text](/img/2024-07-21-CloudControlTestGen/image-1.png)

上述问题要求我们对整个智能系统进行更深入的闭环考核。GPT已经通过了图灵测试，但是对于开车的AI，怎么定义它的“图灵测试”呢？

图灵测试旨在探究机器能否模拟出与人类相似或无法区分的智能。

于是，对车载AI进行定性评价：这个AI开车像人开车。

基于需求设计和实施评价的过程中，测试是连接需求和评价的重要一环。

![alt text](/img/2024-07-21-CloudControlTestGen/image-2.png)

仿真、封闭道路和真实场景的测试时间成本很高。

最大的问题在于，corner case很难复现：比如人在面对特殊情况之前会接管车辆，以及极端情况占了所有case的少数。

要加速测试的过程，就要多搞点corner case。

![alt text](/img/2024-07-21-CloudControlTestGen/image-3.png)

目前业界的典型测试用例如上。30km下创假人和假车。

一个问题在于，这个用例本身有多大参考意义。创了说明车载AI性能不行，不创是不是就说明车载AI性能行？如果把触发停车的阈值设置得特别低的话，会不会在测试表现好，但是在上路时触发更多的误制动？

注意，这个时候还仅仅是安全性问题，离考虑AI是否拟人还早着呢。

![alt text](/img/2024-07-21-CloudControlTestGen/image-4.png)

于是就有上图所示的测试评价的若干问题。

![alt text](/img/2024-07-21-CloudControlTestGen/image-5.png)

为了解决上述问题，提出了智能网联汽车云控多交通参与者测评系统。

![alt text](/img/2024-07-21-CloudControlTestGen/image-6.png)

上图是这个系统的一些亮点：基于5G云控的车路云一体化，大规模交通参与者的协同规划。

![alt text](/img/2024-07-21-CloudControlTestGen/image-7.png)

交通参与者类型多：假人、假车、假场景、假摩托，而且参数全部可调...

![alt text](/img/2024-07-21-CloudControlTestGen/image-8.png)

连续的场景：并不是像现在那种路测刹一把车/创一把车就结束了，而是在持续生成复杂场景。

![alt text](/img/2024-07-21-CloudControlTestGen/image-9.png)

冲突生成：如果愿意的话，可以让模拟的行人/车对被测车充满敌意。

![alt text](/img/2024-07-21-CloudControlTestGen/image-10.png)

架构如上。

![alt text](/img/2024-07-21-CloudControlTestGen/image-11.png)

![alt text](/img/2024-07-21-CloudControlTestGen/image-12.png)

可以基于5G信号进行连接控制，自主设计复杂行车路线。

![alt text](/img/2024-07-21-CloudControlTestGen/image-13.png)

![alt text](/img/2024-07-21-CloudControlTestGen/image-14.png)

![alt text](/img/2024-07-21-CloudControlTestGen/image-15.png)

5G云控做一些修正，做一些可靠控制。

![alt text](/img/2024-07-21-CloudControlTestGen/image-16.png)
