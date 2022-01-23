# Title

Pollux: Co-adaptive Cluster Scheduling  for Goodput-Optimized Deep Learning

## Website
https://arxiv.org/abs/2008.12260

## Citing

@unknown{unknown,
author = {Qiao, Aurick and Neiswanger, Willie and Ho, Qirong and Zhang, Hao and Ganger, Gregory and Xing, Eric},
year = {2020},
month = {08},
pages = {},
title = {Pollux: Co-adaptive Cluster Scheduling for Goodput-Optimized Deep Learning}
}

## Brief Introduction

该论文提出了一种针对深度学习任务实际吞吐率的公式化表达，通过对资源分配与模型参数的协同调整对实际吞吐率进行优化，并提出了基于此的调度器 Pollux。实验验证，Pollux 平均减少了37%-50%的任务完成时间，同时和节点平均分割的集群相比，加速比达到了1.5-5.4倍。

## Key Methodology

- 定义

  - 实际吞吐率 = 吞吐率 * 统计效率

  - 统计效率：在每轮迭代中，模型精度的提升程度。给定初始 batch size *M<sub>0</sub>*，若使用batch size *M* 进行训练相比于 *M<sub>0</sub>* ，需要使用 *1 / E* 倍样例，则对于模型的 batch size 为 *M* 的情况，统计效率为 *E* （0 < E ≤ 1）。

  - 吞吐率：总 batch size（总 GPU 数 * 单 GPU 的 batch size * 梯度累积值）与每轮迭代时间的比值。其中每轮迭代时间 *T<sub>iter</sub>*  可以分解为各节点计算本地梯度时间 *T<sub>grad</sub>* 和所有 GPU 平均梯度与同步参数的时间 *T<sub>sync</sub>* ，考虑到重叠，*T<sub>iter</sub>* 的值在后两者之和与最大值之间。

- 调度器设计

  - 任务层面：PolluxAgent

    - 在任务中持续测量任务的梯度噪声尺度（GNS）与吞吐率，并定期发送给 PolluxSched。并以此结合当前的资源分配，找出效率最高的 batch size，并将学习率缩放至向对应的值。在训练过程中，PolluxAgent 取实际吞吐率最高的 batch size、学习率与资源配置的组合，作为系统的最优配置。

    - 测量每轮迭代时间，记录下其与资源分配、单 GPU batch size、梯度累积步数四者的所有可能组合。每轮迭代都计算统计效率。当统计效率变化时，重新计算系统的最优配置。
    - 参数初始化。根据任务使用的 GPU 数量、节点数量（是否多于一个），设定吞吐率的初始参数，即 GPU 间平均梯度的时间及节点间平均梯度的时间是否为0 。

  - 集群层面：PolluxSched

    - 适应度函数：各任务在当前资源配置下加速比的广义平均数。
    - 加速比：某任务当前资源配置与资源平均配置的情况之比
    - 再分配惩罚：为了防止资源再分配过于频繁，每次再分配之后的加速比需要乘再分配系数（小于1大于0），在任务的生命周期中历史再分配次数越多，该系数越小。
    - 干扰预防：禁止不同任务共享同一节点，避免同一节点上的不同任务相互干扰。
    - 支持参数固定的任务：对于要求参数固定的任务，如固定 batch size，则将其统计效率固定为1，只调整其他可变量。


## Data sets & Experimental Design

实验环境：a cluster consisting of 16 nodes and 64 GPUs

实验数据：deep learning cluster traces published by Microsoft

实验对象：Pollux, Tiresias, Optimus

比较方面：平均 JCT、makespan

实验结果：

- 宏观基准测试

  对于已经预先调整过配置的任务，Pollux 在平均 JCT、makespan 等指标上均优于其他两者。

  对于实际任务配置，Pollux 的优势更加明显。

- 其他测试

  负载敏感度：对于变动的负载，Pollux的平均 JCT 与 makespan 的变化幅度最小。

  调度间隔：间隔越大，平均 JCT 上升。

  同一节点任务间的相互干扰：相互干扰的程度越大，平均 JCT 越大，而禁止将不同任务部署到同一节点上能有效防止 JCT 因相互干扰而增大。

## Concluion And Future Work

该论文提出了实际吞吐率（goodput）的概念，将吞吐率与统计效率进行结合，将硬件资源分配与超参数之间建立依赖，在此基础上提出了调度器 Pollux，为性能优化问题提供了一个启发性的方向。
