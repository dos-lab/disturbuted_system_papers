# Title

Llama: A Heterogeneous & Serverless Framework for Auto-Tuning Video Analytics Pipelines

## Website
https://doi.org/10.1145/3472883.3486972

## Citing

@inproceedings{10.1145/3472883.3486972,
author = {Romero, Francisco and Zhao, Mark and Yadwadkar, Neeraja J. and Kozyrakis, Christos},
title = {Llama: A Heterogeneous &amp; Serverless Framework for Auto-Tuning Video Analytics Pipelines},
year = {2021},
pages = {1–17},
numpages = {17},series = {SoCC '21}
}

## Brief Introduction

该论文提出了一种可用于异构系统的、对视频管道进行自动调优的无服务器框架LLAMA。现有方法需要用户自行选择操作配置并选择相应的物理资源。而LLAMA计算操作中的每个调用的目标时延，在此基础上结合成本在异构资源中选择相应的配置，使其满足调用层面的时延。与现有框架相比，LLAMA平均降低了7.8倍时延与16倍成本。

## Key Methodology

1. Offline specification

   - 应用接口部分

     用户需要说明管道（DAG 的形式）的操作、操作之间的依赖关系以及条件工作流（conditional flow）。操作可以来自 LLAMA 的库，也可以由用户自行定制（关键是 tunalble knobs，比如硬件类型、batch size 等），其中包括了操作的配置选择以及性能参数等信息。输入会添加到元数据库中，并在之后进行复用。

   - 操作分析器（operation-profiler）
   
     对于上一步的配置选择以及性能参数，首先对每种可能的配置进行枚举，分别使用若干采样帧进行测量，记录时延、资源使用情况。由于资源争用或者是测量误差，在系统运行时，操作调用的性能会和之前的测量值不同，此时会根据实际值对测量值进行反馈调整。
   
   - 管道分解（pipeline-decomposer）
   
     将管道（DAG 形式）按照可能路径，使用深度优先搜索进行分解。
   
2. Online

   - 管理器

     接受视频输入以及对应的时延目标，对整个管道的执行进行编排、维护执行状态、生成新的调用。使用指数平滑算法，在一次调用结束后，使用得到的运行数据（时延、成本、配置信息），对之前的测量值进行反馈。

   - 分配器
   
     在操作的调用层面上，对缓冲时间（slack）进行分配，在满足缓冲时间的基础上选择最高效的配置。
   
   - 调度器
     根据分配器，在硬件平台上执行调用，也包括创建、管理后台连接；减轻 straggler；处理失败的调用等、
   
3. 目标时延跟踪的配置策略

   - 估算调用的缓冲时间

     对于给定的调用，计算所有包含该调用的 DAG 剩余路径时延的比例与剩余时间的乘积，取最小值为分配的剩余时间值（简单来说是所有剩余路径中总时延最长的，即最坏情况）

   - 配置目标

     对于一个操作所有可能的配置以及分配的缓冲时间，分别计算单位数据量所需的成本（成本除以 batch size），取最小值。LLAMA 也支持用户自行决定目标函数。注意到在上一步估计缓冲时间时，使用的是最坏情况，而实际上这种情况并不总是存在。由于每个调用分别动态地进行配置，因此早期保守的缓冲时间估计会在接下来的动态配置过程中自行进行纠正。
   
   - 配置重设定
   
     由于后端并行限制，不能将所有的调用同时并发执行。LLAMA 对调用分配了相应的配置之后，将其放入等待队列中。当其被实际调用时，原来的配置设定可能已经成为了次优方案。因为：之前的调用执行时间可能与预测量值不同、该调用的估计时延可能已经被反馈机制调整过、后续调用数可能已经大量增加。对此，LLAMA 对每个后端设定一个无界的推测序列 SQ 以及一个有界提交序列 CQ，第一次设定配置的调用进入 SQ，出队列时，再次分配配置，进入 CQ。
   
   - 操作间的优先级设定
   
     有时调用过少，难以满足配置要求的 batch size，LLAMA 采用 safe delayed batching，会在有足够多的上游调用以及不超出缓冲时间的情况下进行等待，直到满足 batch size。此外，LLAMA 使用基于优先级的提交策略，对于每个操作进行特定数目的调用，并且优先选择管道中层次更深的操作，避免深层的操作因为分配的缓冲时间太少，来不及被正确反馈，然后，对于给定的操作，计算其在特定后端执行与其他后端相比的受益，称为 affinity，取最高值作为优先级。
   
   - Straggler 与失败调用的处理
   
     调度器监测每个调用的执行时间，如果超出阈值，就令管理器创建该调用的副本，重新开始整个调用流程。




## Data sets & Experimental Design

实验环境：GCP（Google Cloud Platform）

基线：集群系统（Scanner & Nexus）、无服务器系统（gg）、目标跟踪系统（GrandSLAm）

比较方面：管道处理时延、成本

实验结果：

1. 不计成本时，LLAMA 时延最短（llama-fast）；不计时延时，LLAMA 成本最低（llama-cheap）
2. 将上一个实验的 llama-fast 与 llama-cheap 的时延与成本的均值作为目标，LLAMA 能满足时延目标，在时延目标较宽裕时，成本水平与 llama-cheap 相同。
3.  禁用反馈机制，会导致超出时延目标；禁用 safe delayed batching 或基于优先级的提交策略，会提高成本支出。
4. 强制使 3% 的调用失败，依然可以完成时延目标。




## Conclusion And Future Work

该论文提出了一种异构的无服务器视频分析管道自动调优框架，和同年该会议的另一篇论文（Scrooge）可以对比来看，离线的 profile 流程可以进行参考。
