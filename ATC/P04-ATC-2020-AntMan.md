# Title

<!-- 此部分是论文标题-->
AntMan: Dynamic Scaling on GPU Clusters for Deep Learning

## Website
<!-- 网址，有DOI的建议用DOI地址-->
https://www.usenix.org/system/files/osdi20-xiao.pdf

## Citing

<!-- 引用格式，建议使用latex格式-->
@inproceedings{xiao2020antman,
  title={AntMan: Dynamic Scaling on $\{$GPU$\}$ Clusters for Deep Learning},
  author={Xiao, Wencong and Ren, Shiru and Li, Yong and Zhang, Yang and Hou, Pengyang and Li, Zhi and Feng, Yihui and Lin, Wei and Jia, Yangqing},
  booktitle={14th $\{$USENIX$\}$ Symposium on Operating Systems Design and Implementation ($\{$OSDI$\}$ 20)},
  pages={533--548},
  year={2020}
}

## Brief Introduction

<!-- 通过三五句话描述这篇文章，包括 1. 论文的应用场景；2. 论文克服已有方法的局限性；3. 论文主要的技术手段； 4. 论文的预期结果 -->
大规模集群的深度学习训练任务调度。之前很多调度器为每个任务分配独占的GPU，但是会浪费大量的GPU资源。论文针对多任务共享GPU的场景，提供了新的原语用来控制一个任务的GPU显存和计算单元使用量，能够使得多个任务占用同一个GPU时，保证其中一个任务不受干扰。这些原语能够细粒度控制一个任务对GPU的使用，开辟新的调度可能性。在调度策略上，将任务分为resource-guarantee的和opportunistic的，前者任务之间不能共享GPU，当前者与后者共享时，将使用新的原语，保障前者类型任务的性能。调度算法比较简单，是非抢占式的，贪婪的分配。

## Key Methodology

<!-- 分点写，论述论文中主要技术手段的实施过程 -->
- 动机

    1. 单一任务利用GPU的效率较低

        只有20%的任务利用了一半以上的显存，只有10%的任务利用了80%以上的计算单元。

    2. gang-schedule的idle waiting很长

        多GPU训练任务，需要gang-scheduling，一个job只有等所有需要的GPU都空闲下来才可以开始训练（gang-scheduling的要求如此，为何不可以暂时先用较少的GPU训练还不清楚）。当一个要求GPU数量较多的任务来到时，会为它先保留几个GPU（否则会产生饥饿，或者要更换为抢占式调度），在这上面等待的idle时间会造成资源浪费。

    3. 动态的资源需求

        如一个DLT任务可能在训练一段时间后会进行validation，这时的GPU利用与训练时可能完全不同。如果固定地将最大的需求显存分配给该任务，则会造成浪费。
    
    4. 对于over-subscription资源的巨大的潜力

        在共享GPU上调度的特点：
        1. 通常DL模型的大小比较小，大部分的GPU显存可以在co-executing的jobs上进行调度。
        2. mini-batch的周期通常比较小，在每个mini-batch的边界，进行细粒度的GPU内存和计算调度。
        3. mini-batch每一轮的计算差不多，可以衡量任务的性能，可以衡量出它们的干扰程度（RNN存疑）

- 在DL框架中，动态调整GPU使用率

    - co-execution的挑战：

        1. 每个job在其最小的资源需求上运行，同时还要避免GPU OOM。

        2. 计算单元使用是波动的，要限制潜在的冲突。

    - GPU内存管理

        使用统一内存访问技术：将tensor从GPU切换到CPU内存。

        在之前的DL框架中，一些张量仅在某些训练的stages被使用，但是这部分缓存的GPU内存没有释放。这种设计优化了单一任务的性能，但是失去了潜在的sharing机会。

        AntMan限制GPU内存使用上限。主动监测使用中的内存，将多申请的用于缓存的内存在有必要时释放掉。

        新的原语能够缩小GPU内存使用的上限，即使它小于GPU内存的实际需求。

        如果要用的内存大于GPU显存使用上限，Tensors可以分配到GPU之外的host内存。这是universal memory的功劳，即使内存不够也可以继续运行。然而我们会检测到它的性能下降。当动态地提高GPU显存利用上限后，在CPU上的tensor又会转移回GPU。

        每个mini-batch会创建大小几乎相同的tensor，所以在mini-batch的边界对任务使用的GPU显存上界进行调整。

        当发生利用GPU显存突增时，会超过设定的GPU显存上限，这时分配器会将这部分tensor放入内存。当mini-batch结束时，能够检测到这轮mini-batch的性能下降，此时通过上调它的GPU显存上限，就可以在后面的mini-batch中能够使它拥有足够的显存。

    - GPU计算单元管理

        每个任务对GPU计算单元的使用能力不同。
        
        现有的DL框架只要当一个算子可以执行时就执行。所以GPU kernel的执行频率没有得到管理。
    
        AntMan：当一个GPU算子准备好执行时，被添加到GpuOpManager，由他来控制执行频率。如果想要让一个任务的算子不干扰另外一个任务，则将它的频率调低，通过观察另外任务的mini-batch速度，得出频率降低多少最为合适。
    
        这样就提供了新的原语来限制一个DLT任务的GPU利用率。

- 协作调度器

    - 分层的调度：

        Global Scheduler在高层，作为集群的调度器，为每个节点分配任务。
        Local Coordinator在每个节点上，使用动态资源扩缩容的原语管理任务的执行。

        将任务分为rosource-guarantee和opportunistic。
        Resource-guarantee任务消费固定数量的GPU配额，保证他们的性能与独占运行一样。
    
        数据统计信息被local coordinator的统计模块收集，然后在集群统计模块聚集这些数据，自底向上的方法，来帮助做调度决定。

        除了硬件信息：GPU利用率，GPU内存使用。
        还利用mini-batch时长，最高显存使用，最小显存使用，以及host内存使用。这些也可以帮助scheduler做调度决定。

        一旦一个任务在GPU服务器上启动了，这个local coordinator就管控它的整个生命周期。不使用抢占式调度。

    - 调度策略

        - 目标

            多租户的公平性是主要目标（公平性这里指是保障DLT任务的SLAs，在保证给定的资源条件下），其次是提高资源利用率。

            先前的调度器放弃了一部分公平性，将多余资源分配给要求资源更多的租户，以提升资源利用率。但是这样的GPU资源如果不通过抢占式调度，很难拿回来。

            不使用抢占式调度是因为将正在运行的任务停止，会消耗大量的GPU时钟周期（这是它的说法，但是抢占式调度具有其他优势）

        - 全局调度器，以及调度算法

            如果是一个resource-guarantee任务，则找到GPU数量足够的空闲节点，分配给它。如果GPU数量不够，则先为它预留（reserved）这部分GPU，等待GPU数量足够再开始训练。

            如果是一个opportunistic任务，则找到GPU数量足够的，且负载最低的节点，直接分配即可。

            resource-guarantee任务在reserved gpu时，虽然这些reserved GPU不会被其他的resource-guarantee任务使用，但是可以被opportunistic任务使用。

            全局调度器将监测每个任务在GPU配给不够时的排队等待时间。
            如果排队时间太长就被自动作为opportunistic任务执行。

        - local coordinator

            Local Coordinator做了什么？
            1. 如何保证resource-guarantee任务在共享执行时的性能。

                当已有oppotunistic任务执行，又来了新的resource-guarantee任务时，首先限制opportunistic任务的GPU显存和计算单元，给resource-guarantee任务以足够的资源初始化模型，直到稳定的执行后，将剩余的GPU显存再分配给oppo任务。
                
                对于计算单元，它在不干扰resource-guarantee任务的情况下，逐步地增加oppo任务的利用率。（检测任务的mini-batch时间来确定是否被干扰）。

                类似的，当oppo任务到来时，在保证不干扰resource-guarantee任务的情况下，逐步地增加资源。

            2. 描述了解决resource-guarantee任务的资源需求突增问题。

                当guarantee任务增加GPU显存需求时，tensor会暂时储存到host内存上，归功于universal memory。

                local coordinator在这时缩小其他oppo任务的显存以及计算单元配额。

                值得一提的是，它采取的是应用级性能量度（mini-batch时间），如果检测到不稳定的性能，采取悲观的策略。

            3. 介绍了一个贪婪方法，当GPU只被opportunistic任务使用时，最大化任务的性能。

                当oppo任务需求的显存超过总量时，使用简单的启发式方法，分配给那个能够增多性能最多的任务以GPU显存。
                
                这是通过一个试错分配过程实施的。

- 实现

    修改pytorch和tensorflow里面的内存分配模块，以及gpu算子执行模块。


## Data sets & Experimental Design

<!-- 撰写实验环境的设置，实验的对象，实验的比较方面，以及实验的结果（不要列举数据，要概括谈） -->
- 实验环境

    小规模k8s集群（64张V100），以及阿里的生产环境（5000个异构GPU）

- 比较方面

    - 对于模块
      - 动态GPU显存扩缩容的效果和效率
      - 动态控制GPU计算单元使用率的效果
    - 端到端
      - 基于真实trace的实验，比较JCT和makespan
      - 在真实生产集群上，比较排队延迟以及resource-guarantee任务的受干扰率。

## Conclusion And Future Work

<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->
提供了对GPU显存和计算单元更加细粒度的控制原语。利用原语，AntMan将集群调度器和DL框架协同设计，合作地来管理DLT任务，使得GPU能够被多个任务使用时，为opportunistic任务提供best-effort的服务，同时又不影响resource-guarantee任务的执行。

这些原语能够使得多个任务共享GPU时，限制某些任务的使用率使得其中一个任务的性能得到保证，但与不使用原语相比，不一定能够提升整体吞吐率。

在显存不足时能够将tensor转移到host内存，能够使得即使内存不足时也能继续运行，这点能够帮助space-sharing的任务的执行。

对于space-sharing的任务调度，这些细粒度的原语提供了更多的探索空间，在将来需要考虑space-sharing时，这里的方法需要着重参考。
