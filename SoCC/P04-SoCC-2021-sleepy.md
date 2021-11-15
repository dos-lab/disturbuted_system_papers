# Title

Tell Me When You Are Sleepy and What May Wake You Up!

## Website
https://doi.org/10.1145/3472883.3487013

## Citing

@inproceedings{10.1145/3472883.3487013,
author = {Mvondo, Djob and Barbalace, Antonio and Tchana, Alain and Muller, Gilles},
title = {Tell Me When You Are Sleepy and What May Wake You Up!},
year = {2021},
booktitle = {Proceedings of the ACM Symposium on Cloud Computing},
pages = {562–569},
numpages = {8},
series = {SoCC '21}
}

## Brief Introduction

在云计算场景中，hypervisor / 宿主机操作系统层的调度器与客机层的调度器，两者无法进行交互，会导致调度混乱的情况，降低 CPU 的时间利用率，具体来讲是低层的调度器令每个微服务时间上平等地在 CPU 上运行，但很多情况下，被调度的微服务处于忙等或缺少输入等情况，使得这部分 CPU 时间被浪费掉了。本文使用实验对该问题进行了验证，在此基础上对传统的hypervisor / 宿主机操作系统层的调度器进行改进，提高了 CPU 的时间利用率并降低延迟，进而提出了可能的应用方向。

## Key Methodology

第一种方式：由可信的来源，如云编排器（Cloud orchestrator），以 manifest file 的形式向服务器及目标虚拟机提供能被 hypervisor / 宿主机操作系统层的调度器所接受的运行中软件的相关上下文信息（包括不同的 FaaS 微服务间，如何调度入链、它们之间交换的信息等）。

第二种方式：与上一种类似，但是是由运行中的虚拟机提供上下文信息，缺点是安全性不足，有可能遭到 DoS 攻击。

第三种方式：调度器不再被动接收上下文信息，而是通过虚拟机自省的方式主动重现，并且通过监视客机来确保安全性。调度器以这种方式来调整与目标虚拟机的交互。

提出了基于 eBPF 的 Linux/KVM 调度器，可以在运行时针对每个可调度的实体（schedulable-entity）进行自适应调整，以监测并判断微服务的睡眠以及唤醒时机。


## Data sets & Experimental Design

实验环境：AWS EC2 a1.metal、in-lab server(PowerEdge R430 with an Intel Xeon E5-2620 v4)，Firecracker、Apache OpenWhisk

实验对象：CPU 过载比，即虚拟CPU（vCPU）与物理CPU（pCPU）的数量比：1:1, 3:1, 6:1

实验的比较方面：

1、不同任务下的函数链执行总时间、平均空闲时间、从触发器激活到开始执行函数链之间的延迟时间

2、不同并发调用数下，平均触发器-函数链延迟、单链总执行时间、微虚机空闲时间比、平均函数间延迟（将下一个函数调度到链中的时间）

3、基于 Apache OpenWhisk 重复以上实验，目的是用容器代替虚拟机。

实验结果：

1、CPU 空闲时间约占总 CPU 时间的 1/5 到1/4 左右。

2、并发调用数（0~50）增加，各项测试内容都显著地增加。

3、在容器环境中，CPU 空闲时间占比与虚拟机环境相比显著下降。


## Conclusion And Future Work

<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->

文章的绝大部分内容是问题提出与问题分析，实验部分也是对问题的阐述。具体方案的提出篇幅很小，而且只有语言性的描述，没有做相关的具体设计以及实验验证，甚至一张图也没有。文章把提出了“软件的多个层在同一物理机上运行且没有交互，因此会导致大量的 CPU 时间浪费，最高能达到69%~75%”这个问题当做是主要的贡献之一。但是感觉实验和问题的关联性好像也不是很大，总感觉中间缺少一点过渡性的论证。
