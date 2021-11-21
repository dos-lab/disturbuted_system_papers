# Title

Tiresias: A GPU Cluster Manager for Distributed Deep Learning

## Citing

Gu J, Chowdhury M, Shin K G, et al. Tiresias: A {GPU} Cluster Manager for Distributed Deep Learning[C]//16th {USENIX} Symposium on Networked Systems Design and Implementation ({NSDI} 19). 2019: 485-500.

## Brief Introduction

针对同构集群DL任务的调度问题，已有工作存在较多问题，kubernetes、yarn的静态调度器过于简单，无法满足实际需要；Optimus等基于DL的建模，其预测训练时长的准确度不高（收敛曲线不够平滑）；基于约束和数据本地性考虑，以最少机器和最少网络通信的方法也未必能够提升性能。Tiresias为了有效的调度GPU，最小化JCT，同时提升efficiency，通过对生产环境的job信息进行分析，面向不同的DL模型，提出两种基本思想：基于离散的2D-LAS和2DGittins-index方法。

## Key Methodology

同时考虑空间约束和时间的不确定性，使用Gittins和LAS两个优先级设置方法，使用多级队列（不同优先级）处理避免饥饿等情况。

- 先前调度方法的问题

    1. 训练时间
       
        先前方法要么认为训练时间可知，要么认为训练时间可以基于它的重复执行特性进行预测，但是DL任务如果以loss作为标准而结束时，先前方法乐观地认为loss的收敛曲线是平滑的。但是事实上并非如此。

    2. 任务放置过于倾向consolidation的。

        之前的算法通常认为要尽可能避免网络数据交换，尽最大可能的变得consolidation。但是tiresias认为这个结论不完全正确。

- 设计

    1. 设计目标即限制
        - 设计目标
          - JCT
          - Utilization
          - Startvation Free
        - 限制
          - job的资源需求由用户和框架决定。
          - 对任务的长度不可知，但是任务长度的分布基于历史数据可能可知。
          - 对job的特性不可知：不知道底层DL框架如何将tensor分配给哪个Parameter Server
          - 对于一个DLT任务，它需要的全部资源都可用时，才能开始。

    2. 整体架构

        三大模块：
        1. Scheduler
        2. Placement
        3. Profiler
          
    
        Scheduler周期性的对全部任务进行调度。刚刚到来的以及被抢占的没有运行的任务放置在等待队列中。当任务到来、任务完成、或资源变更时，会进行一次调度算法。调度算法得到一个任务的调度顺序，将交给Placement把任务放置到集群上运行。
    
    3. 调度算法
    
        为满足设计目标中的三个目标，抢占式调度是需要的，因为需要解决队头阻塞的问题。抢占式调度的一般有time-sharing，SJF以及SRTF三大方法。time-sharing如Gandiva，目标并不是减少JCT。而SJF和SRTF需要知道准确的任务执行时间。
    
        - 为何需要考虑时间和空间两个维度？
          
            本文设计了既考虑时间又考虑空间的启发式方法。设计了SRSF方法：shortest-remaining-service-first。用任务的剩余时间乘以它需要的GPU数量作为度量。
    
            SRSF分别与SF（使用最少GPU优先）以及SRTF进行对比。发现SRSF通常更优。这说明既考虑任务剩余时间又考虑任务需要的GPU数量这两个维度，一般能够带来更好的结果。
    
        - 2维的 attained service-based调度算法（2DAS)
    
            2DAS在不依赖精确地执行时间情况下，将GPU需求数量考虑进来。
    
            它将经典的least-attained service（LAS）以及Gittins index policy针对DL模型训练场景进行泛化，得到针对DL任务的，并且能够同时考虑时间和空间的调度算法。
    
            任务i的attained service是基于以下计算的：
            它使用的GPU数量($w_i$)乘以它已经运行的时间长度($t_i$)。该乘积越大，则说明它获得的服务越多。
    
            当一个任务的长度分布不可知时，使用LAS确定优先级：即优先级为它attained service的倒数。
    
            如果知道任务长度的分布，优先级通过使用Gittins index公式计算。这个公式体现了这个任务在将来得到一定量的服务后，有多么大的可能会任务完成。该值更高则优先级更高。
    
        - 离散的多级调度队列
    
            虽然使用2DLAS能够得到优先级，但是它得到的是在实数域上的连续的优先级。如果使用连续优先级做抢占式调度，则：
            1. 不断地抢占造成的overhead成本太高。
            2. 2DAS将会降级成为一个公平的基于时间多路复用的算法，这会增加JCT。
    
            本文将不同优先级划分成K个区间，作为K个队列来考虑。这就将优先级作为离散变量来考虑了。从效果来看，它将attained service相似的job放在了同一个队列里。
    
            随着时间推移，每个job的attained service变化，它们有可能在队列之间移动。
    
            - 如何决定队列个数K
              > 在决定K的大小以及queue的阈值上，采取了经典的 前台-后台 队列思路，已被证明在长尾分布上更有效。只有两个queue，和一个阈值。
              K = 2与K更大时，如果忽略抢占overhead是差不多的，但是K = 2时抢占更少，会更好。
              （为什么K越小抢占越小：因为K越小，则每个队列的任务越多，则每个任务更有可能维持在同一个队列不动，只要任务还处于同一个队列，就不会被抢占调度） 
    
            - 避免饥饿
    
                当一个任务的等待时间太长，直接将它放置到最优先队列中。通过一个参数可以调整对于饥饿调度的敏感性。
    
        - job placement
    
            为保证数据本地性，它首先尝试使用ILP对网络带宽进行最优化分析。该ILP考虑为：当一个新的job加入集群后，最小化网络traffic。但是在实验后发现有问题：1. 解ILP很慢。2. 在小规模集群上验证，发现并不一定能提升性能。
    
            于是转而思考：哪个任务对consolidation更敏感。
    
            通过实验得出结论：发现拥有超大tensor的DL模型对consolidation更敏感。这是因为message size跟模型的结构紧密相关。如TensorFlow框架会将每个Tensor作为一个数据通讯包（或分批），这时，越大的tensor会导致带宽占用越高。
    
            所以尝试通过profiler，从网络库底层监控每个job在每个GPU worker上的通讯量（包括GPU link以及Network），来估计它对consolidation有多敏感。
    
            得出一个值来估计这个敏感度，高于阈值则认为是敏感的，在做放置时更倾向于consolidation，而低于阈值的，在放置时就尽可能减少集群的fragmentation。阈值的选择采取了简单的线性分类器。
    
        

## Data Sets

wordload没有开源的数据集，
cnn模型选择TensorFlow官方的模型（https://github.com/tensorflow/benchmarks）

## Experimental Design

1. 基准测试：yarn
2. 衡量指标，JCT的缩小比例
3. 实验环境：集群环境与模拟统计的工作负载（480个作业的训练时间和资源使用量）

## Conclusion and Future Work

在JCT这一个指标上有显著提升，其他指标也许无法保障，对DL的粒度控制的也相对较粗。
