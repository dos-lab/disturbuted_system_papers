# Title

Chronus: A Novel Deadline-aware Scheduler for Deep Learning Training Jobs

## Website

https://doi.org/10.1145/3472883.3486978 

## Citing

<!-- 引用格式，建议使用latex格式-->
@inproceedings{gao2021chronus,
  title={Chronus: A Novel Deadline-aware Scheduler for Deep Learning Training Jobs},
  author={Gao, Wei and Ye, Zhisheng and Sun, Peng and Wen, Yonggang and Zhang, Tianwei},
  booktitle={Proceedings of the ACM Symposium on Cloud Computing},
  pages={609--623},
  year={2021}
}

## Brief Introduction

<!-- 通过三五句话描述这篇文章，包括 1. 论文的应用场景；2. 论文克服已有方法的局限性；3. 论文主要的技术手段； 4. 论文的预期结果 -->
本文主要针对同构场景下，大规模GPU集群的深度学习训练任务调度。其特定的调度目标为，尽量同时保障SLO和best-effort类型的任务的调度。现实系统中通常一个集群可能被商用和研究用，商用比较注重SLO，研究用则不对DDL有强烈要求，但也注重它的调度延迟。以往的工作大多不能兼顾这两点。本文系统将在集群层面将这两类任务隔离，保证各自拥有足够的配额。通过profiler预测任务执行时间，通过一个选择器进行准入控制和任务选择，在这其中，将能够保证在DDL前完成的SLO任务划分到一个集群，剩余的SLO以及best-effort划分到另外的集群。在SLO保证能完成的这批任务中，通过MILP（混合整数线性规划），选择一批优先级高的任务执行。将每个任务划分成固定的租约，当租约到期重新使用MILP选择任务，实现抢占式调度。在另外的集群使用SRTF的方式调度。在分配任务到集群中时，考虑到资源位置亲和性，使用round-up以及原创的local space search方式，保障任务尽可能执行在consolidation的GPUs中。

能够大幅减少SLO违反，并且保证best-effort任务的JCT。

## Key Methodology

<!-- 分点写，论述论文中主要技术手段的实施过程 -->
1. 工作流简述

    工作流简述：
      1. 使用Profiler，筛选出短期的任务或有问题的任务。测量并预测长期任务的运行时间。
      2. 当profiling结束，Selector执行准入控制。为每个SLO任务打标签，guaranteed标签表示能够达成DDL，unguaranteed表示DDL很难达成。
      3. 对于guaranteed任务，使用MILP求解器，识别需要调度的任务。
      4. 对于unguaranteed或best-effort任务，使用SRTF（最短剩余时间优先）算法来选择任务。
      5. 一个Allocator用于分配选择的任务以GPU。

        对于guaranteed任务，采取round-up技术来发现consolidation的GPU。

        对于其他被SRTF挑选出来的任务，采取local search算法找出一个有效的放置。

2. profiler

    - 提交控制
  
        短时间内大量训练任务到来，可能会导致profiler的拥堵排队。

        使用提交控制策略，限制每个用户在一定时间段内能提交的最大任务数。

    - 减少profiling需要的资源

        之前的profiler通常使用需求GPU数量个数的GPU来做profile。
        
        现在为节省资源，最多使用2个节点的资源来预测分布式训练速度。

        在
        > Deep Learning Research and Development Platform: Characterizing and Scheduling with QoS Guarantees on GPU Clusters.

      中，它分析了对于一个有n个GPU节点的分布式DLT任务，每次迭代的执行时间可以用$t_n = t_c + \log_2{(n)}t_o$表示，其中$t_c$是计算时间，$t_o$是通讯时间。这个公式的意思是说，任务的并行个数已经确定，当只有1个节点时，是consolidation放置的，应该最快。当变为2个节点时，各自节点放置一半并行的model，时间就会需要加上来回传递梯度的时间，即$t_o$。该公式给出了任意节点数下，GPU同构时，且网络带宽一致时，计算时间的公式。（需要看下这篇文章，看下这个公式是否有办法扩展到异构且网络带宽不一致的情况）。
      
      本文使用它的建模预测任意个数GPU扩容时的执行时间（资源同构的）

    - 动态可调整的profiler资源使用。

        将profiler集群建模为一个排队系统。平均任务到达率为$\lambda$，每个任务要服务的时间为固定的$T_{profile}$，计算平均消耗显卡数量$G$。通过排队论数学建模，测量过去一小时的任务达到率，计算出一个使用GPU数量的下界。（这里偏数学，我不太会）

3. Job选择器

    1. 对SLO需求的建模

        使用分段函数表达出一个回报函数。具体来讲，横轴为任务完成时间，纵轴即为在这一时刻完成的回报值。回报值越大越好，作为后面MILP优化的目标。对于严格的SLO，分段函数是两段的，在DDL前全部为100，在DDL后只有1。而对于soft的SLO，这个分段函数在DDL之后还有多个阶梯，表示稍微违反一点SLO也是可以接受的。对于best-effort任务，全部都是1。 

    2. 准入控制

        恶意用户可能指定一个很短的SLO，导致系统一直执行它的任务。系统将调用MILP算法，找到那些集群无论如何也达不到SLO目标的任务，直接放到与best-effort相同集群，采用SRTF调度。避免资源抢占。

    3. 基于租约的训练。

        基于租约的训练，本质还是时间片。在这里分了两个集群，guaranteed集群和非guaranteed集群。他们的租约时间是固定的。

        其中为方便管理，guaranteed集群的lease时间是另一个集群的整数倍。具体原因论文描述的不太明确。

        每当租约过期时，重新进行job的选择调度等。其实与时间片差距不大。

    4. 选择可保证完成的SLO任务。
   
        给出了一个目标函数，其保证最后的调度结果，其对应的回报函数的和最大。使用MILP求解器计算结果。

        在这里，租约长度非常关键。过短会导致太多抢占和MILP的求解延迟，太长会导致调度灵活度下降。这里还是采取了基于经验的选择了20分钟。我认为局限性很强。

    5. 选择best-effort以及unguaranteed SLO任务

        使用SRTF。当SLO集群的任务执行完，但是租约还有很久时，采用了简单暴力的方法：直接将这个完成的任务使用的资源从SLO集群分配给best-effort及unguaranteed SLO任务集群。

4. 任务分配器

    任务分配器接受一批任务，并将它们分配到集群中。由于DLT任务资源分配位置的敏感性，这里需要考虑尽可能将它们放置的足够“接近”。

    它将任务分为了“集中友好”和“非集中友好”的两类。任务指定的并行数量为$2^n$的为友好型的。它证明了当一个集群的每个节点的GPU数量都是$2^n$时，以及每个任务都是友好型的时候，必定能够找到一个全部都是集中放置的结果。并总结成一个放置算法。

    所以它采取了round-up策略，当来了一个非友好型的job时，提高它的并行数量把它变成一个友好型的（很粗暴）。然后就可以使用上面的最佳集中放置安排。如果集群的中某个节点的GPU数量不是$2^n$，则将它分成逻辑上的多个节点，保证满足要求就能使用以上算法。（同样很粗暴）

    但是对于best-effort集群，只针对要求GPU数量大于16的任务使用round-up方法。当任务要求较少的GPU数量时，使用round-up可能导致某些任务GPU数量膨胀，导致某些任务无法分配得到GPU，从而增大延迟。所以它原创了一种Local search placement算法。它通过profile知道每种放置方案下的计算速度，使用当前放置方案与最优放置方案的运行速度比值，乘以显卡数量，定义一个放置方案的收益。需要显卡数量越多，则任意一个GPU worker时间节省带来的收益更大。乘以GPU数量是必要的。将最大的收益减去最小收益得到该任务的“性能提升潜力”。将任务用该指标排序，对该排序后的列表按照一定阈值选出前k个任务，对这k个任务穷尽他们的放置方案，对它们的全部配置可能进行遍历，每遍历一个配置方案时，剩余的N-k个任务就使用“准-集中”的方式放置，这个“准-集中”的方案是贪婪的，很快能够得到。得到最终放置方案后算出它的收益，遍历结束后，收益最大的方案就是结果。


## Data sets & Experimental Design

<!-- 撰写实验环境的设置，实验的对象，实验的比较方面，以及实验的结果（不要列举数据，要概括谈） -->
- 数据集
  - 来自SenseTime的一个生产环境集群的Helios trace
  - 来自微软数据中心的Philly trace

- 实验设备

    分别在两个同构集群测试，96个节点和120个节点，每个节点8个GPU。

- 实验主要指标

    - Weighted Deadline Miss Rate 用来测量SLO需求
    - JCT 基本只用来测量best-effort任务的JCT

- 按模块测量

    profiler, selector, allocator

- 端到端的测量

    针对wdmr指标，针对不同workload，不同任务提交密度下测试。针对jct指标，只关注了best-effort任务，也在不同workload和不同提交密度下进行了测试。

    这里只测了它优势的一方面，取巧。

## Conclusion And Future Work

<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->
1. 扩展到异构集群
2. 扩展到自动扩缩容的DLT任务，我认为这类弹性的系统应该是未来的趋势。
3. 对DAG任务的调度支持。

本系统是在一系列假设限定条件下设计的：

1. 一个DLT当运行固定数量的iteration后认定为结束。
2. 每个GPU都能放下每个模型。
3. 不考虑模型并行。
4. 针对同构集群。

这些都是将来的系统应该解决的问题。一个DLT的结束可以是由某些条件检测达成（如loss的大小），而模型并行在目前的调度系统还一般无法支持。本系统也没有考虑GPU的space sharing。同时DLT分布式任务的位置敏感性通常很难考虑到网络和服务器的异构拓扑。在这种异构情况下，数据并行训练速度的估计也存在困难。

本文的local search placement方案存在一定参考价值，可以思考如何在异构环境中设计类似的算法。