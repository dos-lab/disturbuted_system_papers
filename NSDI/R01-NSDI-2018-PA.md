# Titile

Performance analysis of cloud applications

## Citing

Ardelean, Dan, Amer Diwan, and Chandra Erdman. "Performance analysis of cloud applications." 15th {USENIX} Symposium on Networked Systems Design and Implementation ({NSDI} 18). 2018.

## Brief introduction

本文通过对Google内部集群上Gmail的应用情况分析（一周数据），从以下几个关键指标上来度量云环境下大型分布式系统的性能：
1. QPS：每秒交易数量；
2. CoR：请求负载特征；
3. UVR 用户可见请求量；


## Key Methology

构造相关图形，进行具体分析，得出用户请求持续变化的结论，得出以下关键结论：
1. 用户请求不是造成负载波动的唯一原因，有很多开销是后台自行运行的（突发事件，定期运维任务等）；
2. 没有很好的方法能够完全复现用户的请求，尽管有效果；
3. 在真实的环境中去测试具有一定的可行性（不能复现真实的负载情况）；

基于请求延迟的概率分布进行性能分析，相应延迟有四倍以上的性能差异，这种变化负载是否提升或者降低延迟：
1. 在不同的用户数量，99%，95%，50%不同比例的用户下，对应的延迟概率分布（一周）。
2. 通过统计方法去判断负载变化的影响，使用K-S检验对正态分布和近似正态分布的数据进行分析（资源利用率预测）。

判断负载变化的影响程度与开销效果：
1. OPS并不完全和相应延迟正相关；
2. 通过trace跟踪的方式进行水平（bursty tracing）和垂直注入（系统事件与应用事件）

## Data sets

没有使用开源数据集，主要聚焦于Gmail的运行时情况

## Experimental Design

1. 比较用户请求，资源利用率、请求延迟等多个指标之间的关联关系；
2. 根据数据请求的分布规律判断相关因素（假期，雷击等）造成的影响；



## Conclusion And Future Work

1. 任务请求资源分布、容器使用资源分布；
2. 从若干维度分析开源数据集，得到机器、任务、实例、和容器相互之间的关系；