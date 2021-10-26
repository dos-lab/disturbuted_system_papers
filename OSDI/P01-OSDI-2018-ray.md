# Titile

Ray: A Distributed Framework for Emerging AI Applications

## Citing
Moritz, Philipp, et al. "Ray: A Distributed Framework for Emerging {AI} Applications." 13th {USENIX} Symposium on Operating Systems Design and Implementation ({OSDI} 18). 2018.

## Brief introduction

针对强化学习场景（通过学习特定的策略来实现问题和答案的映射），其中通过模拟来评估各类策略，通过分布式训练去提升特定策略，这对相关的RL系统提出新的挑战：
1. 必须能够支撑细粒度的计算；
2. 必须能够支撑异构任务（时间与资源空间上）；
3. 必须能够支撑动态任务执行；

综上，Ray是一个动态计算框架，在亚秒级延迟下支持上百个任务的调度管理（不同于BigData工作负载）。


## Key Methology
其出发点：RL应用各类复杂的workload：策略学习，策略服务和策略模拟评估三个过程中面临各类不同的要求：
1. 细粒度的异构计算；
2. 动态执行轨迹；
3. 灵活的计算模型。

Ray的编程与计算模型：
1. Task代表无状态计算；
2. Actor代表有状态计算；
3. 通过wait和get实现同步；
4. 动态图计算模型实现整体计算。

体系结构（系统层面）：
1. 整体控制存储：通过键值对的形式存放所有的系统状态，通过分片实现伸缩和容错，将任务分发和认读调度分离；
2. 冒泡分布式调度：任务首先在本机进行调度，如果负载超限，由全局调度器实现选择再调度；
3. 基于分布式内存的对象存储，所有task的输入输出信息均在mem中存放，通过副本机制实现共享；

体系结构（应用层面）：
1. 通过远程执行实现分布式计算：Driver定义任务，通过每个work上的对象存储和本地调度器实现数据传递；
2. 远程计算完成后，在worker节点上将信息写入，在整体控制存储中更新信息，并通知Driver所在的机器，显示反馈的结果。

## Data sets


## Experimental Design

1. 调度延迟、伸缩性和容错方面的表现；
2. Ray API的额外开销；
3. 在训练、服务和模拟三种工作负载下的Ray的表现（与其他框架相比）；
4. 使用Ray做RL应用时的优势


## Conclusion And Future Work
