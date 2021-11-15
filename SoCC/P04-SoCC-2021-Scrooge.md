# Title

Scrooge: A Cost-Effective Deep Learning Inference System

## Website
https://doi.org/10.1145/3472883.3486993

## Citing

@inproceedings{10.1145/3472883.3486993,
author = {Hu, Yitao and Ghosh, Rajrup and Govindan, Ramesh},
title = {Scrooge: A Cost-Effective Deep Learning Inference System},
year = {2021},
booktitle = {Proceedings of the ACM Symposium on Cloud Computing},
pages = {624–638},
numpages = {15},
series = {SoCC '21}
}

## Brief Introduction

对于基于云计算 & 机器学习模型的媒体应用场景（处理对象为实时音视频流），需要在处理动态变化的输入内容时满足吞吐率与延迟，并最大限度地降低成本。该论文提出了一种将提供媒体应用作为服务的系统 Scrooge，使用分批处理的时分复用以及与空分复用结合的方式，将计算操作以 media DAG (mDAG) 的形式打包到 GPU 云虚拟机中，提高 GPU 利用率。在此基础上寻找满足吞吐率与延迟需求的最低成本策略：将虚拟机种类的分配与虚拟机实例的放置两个步骤进行解耦，分别选择优化策略。同时，通过对模型的平均处理时间与平均输入速率进行监测，并在两者发生显著变化时调用分配算法，从而实现对输入复杂度动态变化的反馈。实验证明，Scrooge 能在延迟达标率超过98%的前提下降低16-32%的服务器成本。

## Key Methodology

1、高效的 mDAG 包装：① 分批处理的时分复用与 ② 空分复用的结合。

① 分批处理的时分复用：使用划分时间片的方式共享 GPU 资源，使用轮询调度策略。在此基础上，为了提高资源的利用率，将多个不同的输入分成 batch ，组合到单一的请求中，以此减少 I/O 请求，提高吞吐率。具体的打包方式：将来自不同客户端、有着不同输入速率，但是有着相同 SLO 的若干流组合为一个 session。Scrooge 假定 session 中每个客户端的输入速率是固定的。每个客户端可以随时加入、离开 session，也会导致一个 session 的总输入率的变化。

② 空分复用：使用 CUDA 流，以在 GPU 上并行执行不同的 DL 任务。

对于输入复杂度的观测：由于 RNN 模型的输入序列与输出序列不一定等长，因此在由多个 RNN 模型组成的应用中，前后模型之间平均的输入元素数量比例会随着客户端流的输入复杂度变化而不断变化，因此会导致模型处理时间发生变化，文中称之为 ballooning，并把模型处理时间随输入复杂度发生变化的关系用 ballooning factor 来表示。

通过以上两种方式，Scrooge 建立了在给定 worker、给定 mDAG 模型下的、表示吞吐率与延迟两者相互关系的经验模型。对于 CPU 模型，只需要测试在单一输入下的模型平均处理延迟。而对于 GPU 模型，吞吐率与延迟的关系受到 batch 尺寸与空间复用程度（以下称并发度）的影响。以往的工作表明，吞吐率在一定范围内随着并发度线性增长，然后当并发度过高引起冲突后下降。Scrooge 将该变点视为 GPU 的可用性能。由于已知 batch 尺寸越大，延迟越高。因此，在保证最坏情况延迟满足 SLO (end-to-end latency objective) 前提下使吞吐率达到最高的 batch size 与 并发度的组合。

2、Allocation，即虚拟机类型的分配算法

以往工作表明，模型对输入率的使用比即为对 worker 可用性能的使用比。在此基础上，对于部分占用的 worker，按照使用比例计算其成本。因此，可使用混合整数线性规划，目标函数即为所有实例的利用率与成本乘积之和的最小值，而约束条件有三个：1、使实际输入速率与虚拟机实例所能处理的输入速率相等。2、对于 mDAG 的两个相邻结点 m > n，从起始节点到达 n 的最大延迟要大于等于从起始节点到 m 的最大延迟与从 m 到 n 的最坏情况延迟之和。3、整个 mDAG 的最大延迟要满足 SLO。

3、Placement，虚拟机实例的放置算法

假定云服务是弹性的，一般情况下，在一个 mDAG 服务中，会完整使用 k-1 个实例，并部分使用 1 个实例。对于完整使用的实例，Scrooge 在 Allocation 中分配的虚拟机类型中寻找未使用的实例进行放置，如果无法进行放置，则排除该类虚拟机，重新进行分配。对于部分使用的实例，由于一个 mDAG 服务会向若干个不同的 mDAG 提供服务，每个 mDAG 都会有一个部分使用的实例，因此会根据其对输入率使用比来将不同的 mDAG 的部分使用实例进行组合放置。如果无法进行组合，则放置在未使用的实例上。

当客户端加入或离开 session、以及其他原因造成输入复杂度的显著变化时，Scrooge 会调用 Allocation 与 Placement 算法。这个过程中，由于 worker 可能会发生变化，以及 worker 自身可能出现故障，因此使用 NFV(network function virtualization)对客户端流进行重路由、状态的迁移。

4、对输入复杂度变化的反馈

Scrooge 的每个 worker 会持续监测模型的平均执行时间与平均输入率，发生显著变化时通知协调器，协调器使用观测到的执行时间与输入率，作为新的输入参数调用 allocation 算法（有可能调用 placement 算法）。

5、其他优化

① 为了避免使用 JIT 编译方式在前几帧带来的延迟影响，Scrooge 的 placement 算法倾向于使用已经载入模型的 worker，且如果有 worker 成为了未被使用的状态，会处于一段时间的“热身”状态，之后再返还给云服务。

② 模型流水线化。一个 GPU 模块经常需要若干个 CPU 操作来进行输入输出的序列化处理以及输入序列的整形操作。因此将 CPU 的操作与 GPU 的计算操作进行流水线化处理。

③ 早期丢包。随着负载的增加，输入会在模型输入序列中进行等待。在处理序列中的输入之前，worker 会测试执行延迟是否满足 SLO，如果不满足的话会直接将其丢弃。


## Data sets & Experimental Design

实验环境：Microsoft Azure 的集群，包括P100 * 8 & V100 * 8， vCPU * 6

实验数据：EarthCam 2021、Realtime action recognition 2019 等

1、与 Nexus 的对比

实验对象：Nexus、Scrooge

比较方面：提供媒体应用的所需费用（成本）；集群中每个 session 的完成率，定义为 1 秒的观测窗中满足 SLO 的帧数与发送的总帧数的比值。

实验结果：在保持完成率在同一水平时，Nexus 的成本比 Scrooge 高16-32%。

2、MILP 与其他算法的对比

实验对象：MILP、暴力搜索、贪婪启发式搜索

比较方面：Allocation 算法的执行时间、与 MILP 的成本比率

实验结果：暴力搜索无法满足实时的延迟要求，贪婪启发式搜索只能得到成本上的次优解，另外，更少的严格 SLO 要求会降低成本。

3、空分复用的作用

实验对象：Scrooge、无空分复用的Scrooge(Scrooge-ns)

比较方面：成本比、吞吐率

实验结果：在不同任务中的不同 SLO 下，ns 版本的成本均高于 Scrooge。对于不同模型，ns 版本的吞吐率均较低。


## Conclusion And Future Work

文中似乎没有过多讨论每次重新调用 Allocation 与 Placement 算法时所需要的额外时间开销。也许用抢占式的方式能改善这个问题，但是不能保证是 cost 最优的方案，总之还是有讨论的空间的。
