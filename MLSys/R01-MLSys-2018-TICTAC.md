# Title

TICTAC: ACCELERATING DISTRIBUTED DEEP LEARNING WITH COMMUNICATION SCHEDULING

## Citing

Hashemi S H, Jyothi S A, Campbell R H. TicTac: Accelerating Distributed Deep Learning with Communication Scheduling[J]. arXiv preprint arXiv:1803.03288, 2018.

## Brief Introduction

本文主要思路如下：
1. 针对分布式深度学习计算过程中的一次迭代过程中通信和计算两种基本情况出发，针对参数服务器环境，分析其中存在的相互干扰（overlap）导致的吞吐量下降，训练效率低等问题；
2. 提出通信的相互依赖关系，通过基础的权重赋值，根据DAG图调整任务调度的次序，最大化重叠系数
3. 注明基于DAG的优化只能针对特定的深度学习系统：一般图处理系统，图本身就是要待处理的数据；流处理系统，允许刘淑娴的概念，通过图表达处理元素的基本过程；深度学习中的图代表输入数据需要完成的计算；
4. 为了平衡通信和计算，一般通过增大每次迭代的批处理数量来提升计算时间；或者降低通信时间，通过推数据、拉数据等network层面和算法层面的优化降低通信开销；
5. 通过TensorFlow内部DAG本身的优化，来调整调度的顺序，成为一种基本的可能。

根据上述的研究，通过抽象出调度的关键元素，实现基本的基于通信和计算的调度优化：
1. 调度问题的定义和输入输出
2. 重叠系数


## Key Methodology

关注的启发式因素：
1. dep，通信依赖：op能够运行时的recv依赖关系，可以通过深度优先遍历；
2. M，通信时间：每个op操作时需要等待的通信时间；
3. P，直接依赖计算负载：每个recv单独操作可以带来的直接计算负载（通过时间计算）；
4. M^+，即将发生的通信负载：激活当前op需要的通信负载和已经产生的通信负载（依赖和产生的都算）

然后本质上是一个优先级的抢占问题：

TIC:
1. 只根据DAG图中的依赖信息作出优先级调整：
2. 通过优先级排序最小化M,计算出调度顺序；

TAC：
1. 根据依赖关系和运行时间统筹考虑优先级；
2. M，P，M^+都考虑。

## Data Sets

Inception v1，VGG-19等开源图片benchmark

## Experimental Design

1. 比较TIC和基准方法（无序调度）在多次迭代次数下的收敛速度，并没有什么太大区别，说明基于优先级的排序方法不影响实际使用；
2. 伸缩性与吞吐量-基于工作节点扩充时候的加速比，（训练与执行）
3. 伸缩性与吞吐量-基于每次batch的size，（训练与执行）
4. 计算与通信重叠情况：OPS基本与吞吐量呈线性增长；
5. 计算与通信重叠情况：重叠系数与美的迭代执行事件的分布关系，系统数越大，执行时间越短；
6. 随着Ops数量的增长，stragger任务出现的数量呈下降趋势

## Conclusion and Future Work

本质是通过TIC和TAC两个核心策略动态调整优先级，需要事先知道OPS和recv执行的开销与耗时，本质上是队列的重排序策略，这些工作存在很多不足：

1. 除PS结构外，能否在其他架构中中发现此种问题；
2. 未看到cluster efficiency和data locality的情况；
3. 多维资源约束的情况。
