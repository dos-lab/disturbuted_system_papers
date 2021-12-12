# Title

<!-- 此部分是论文标题-->
HiveD: Sharing a GPU Cluster for Deep Learning with Guarantees

## Website
<!-- 网址，有DOI的建议用DOI地址-->
https://www.usenix.org/conference/osdi20/presentation/zhao-hanyu

## Citing

<!-- 引用格式，建议使用latex格式-->
@inproceedings{zhao2020hived,
  title={Hived: sharing a $\{$GPU$\}$ cluster for deep learning with guarantees},
  author={Zhao, Hanyu and Han, Zhenhua and Yang, Zhi and Zhang, Quanlu and Yang, Fan and Zhou, Lidong and Yang, Mao and Lau, Francis CM and Wang, Yuqi and Xiong, Yifan and others},
  booktitle={14th $\{$USENIX$\}$ symposium on operating systems design and implementation ($\{$OSDI$\}$ 20)},
  pages={515--532},
  year={2020}
}

## Brief Introduction

<!-- 通过三五句话描述这篇文章，包括 1. 论文的应用场景；2. 论文克服已有方法的局限性；3. 论文主要的技术手段； 4. 论文的预期结果 -->
深度学习训练任务的集群资源分配。本文提出目前的集群资源分配通常存在着sharing anomaly现象，这使得租户相比于一个相同GPU容量的私人集群中，在共享集群中会遭受更严重的排队延迟和性能下降。本文提出了虚拟的私有集群（VC），并考虑到深度学习训练任务需要的GPU亲和性，将集群按照GPU亲和度划分成多级的cell结构。通过buddy cell allocation算法，能够避免sharing anomaly现象。本系统为其他任何一种调度器提供了一个虚拟的集群视图，能够与任何一种调度器协同使用。最终能够消除sharing anomaly的现象发生。

## Key Methodology

<!-- 分点写，论述论文中主要技术手段的实施过程 -->
1. 动机

    现有的集群资源预定机制通常为每个租户划分一个固定的配额（quota），也就是GPU的数量。但是quota难以体现深度学习任务需求的GPU亲和度（placement-sensitivity）。如：某租户有64GPU的quota。但是它希望按照8x8的方式进行分布式训练，但是这64个GPU都散落在各个节点上，无法按照8x8的方式训练。所以只能等待，或者按照64x1的方式进行训练，这会造成排队延迟或性能下降。

    sharing-anomaly是什么？

    sharing anomaly的更精确定义：
    如果一个租户的一个训练任务的请求序列，它的要求不能在shared集群满足，而是能在私有集群满足。（私有集群GPU数量等于quota）。

    就是说使用私有集群比使用sharing集群的效率更高，排队更少，那就产生了sharing anomaly。

    一种减少anomaly的方法就是设计调度策略来减少全局的资源碎片。
    但是这会让调度器更加复杂，因为它还需要管理其他的优化目标。

    比如最小化碎片可能会降低job的性能。因为这增大了job之间的干扰。

    所以本文提出了将对sharing anomaly的解决方案与其他的资源分配目标分离开。

    本文提供的资源保留框架能够与任何的调度器协同工作。资源保留框架专注于消除sharing anomaly。而其他的调度器专注于它自己的调度策略，如JCT，fairness等。

2. 设计

    - 由cell组成的虚拟私有集群

        HiveD将GPU资源按照量层次进行划分，分别是虚拟私有集群（VC）以及物理集群。每个VC由一批预先分配好的逻辑Cell组成。Cell一共只有几种等级，按照它在集群中的亲和度进行划分。后面详细讲。

        这就是说，用户不再指定它需要的GPU配额（quota），而是指定一个VC（它包含了多个各种等级的cell，每个Cell包含一个或多个GPU）。

        VC中的cell是逻辑上的。当一个job使用了一个logical cell的GPU，则该逻辑上的cell会与物理集群上的物理cell绑定。而当没有GPU被使用时，该logical cell会与物理集群解开联系。

        在文中的集群例子中，cell有4个等级：
        1. 单个GPU组成一个1级cell
        2. 在同一个PCIe switch下的GPU组成一个2级cell
        3. 在同一个CPU socket下的GPU组成一个3级cell
        4. 在同一个单个物理节点下的GPU组成一个4级cell

        给cell划分等级后，每个VC的定义就显而易见了：即，该租户在每个层次的cell的需求数量，则可以构成一个虚拟集群。

        构建了这种VC之后，实际上提供了一个集群的视图。任何一种调度器可以在它之上进行调度。

        调度器在调度时，可以随机使用cell内的GPU，即使不是恰好使用cell内全部GPU也没问题。因为这是调度器的决策，与资源预留框架无关。cell是VC与物理集群的分配粒度，而与在VC上层工作的调度器无关。

        在cell的层级关系中，k级的cell由一批k-1级的cell组成。这一批cell在本文就称为buddy cells。
    
        Buddy cells能够组成一个更高级的cell。

        本文假设：每个k级cell能够分割成相同数量的k-1级cell。（这实际上要求集群是同构的，比如一个pcie switch连接4个GPU，另一个连接2个GPU，那就不对了）

        所以，本文在针对异构集群时，将他们划分成多个同构的子集群来考虑。

        在初始时，需要为每个用户的VC分配一个到物理集群cell的1对1的映射。如果在这一步能够满足，就说该VC的分配是可行的。所以，在这里能够看出本文的方法不能支持over subscription。

    - Buddy Cell Allocation Algorithm

        HiveD使用buddy cell allocation算法，管理逻辑cell与物理cell的动态绑定。
        
        它为每个VC维护信息：
        1. 每个分配的逻辑cell绑定的物理cell
        2. 全局上，每个级别的cell的物理上未分配的可用列表。

        该算法总是保留可用的cell的最高层级。即：如果k-1级的buddy cells足够组成k级cell，只有k级的cell被记录。目标是尽可能保留更多的高等级cell。

        该算法实际上是一个递归的分裂过程。当要分配一个k级cell时，如果存在k级cell为空，则直接分配，否则向上，找比k级更大的cell。找到后，将该cell进行分裂，分裂出多个更低级的cell。从中选择一个cell再迭代上面的过程。直到产出一个k级的cell。

        该算法同时使用自底向上的方式解决cell释放。实际上与分裂相反，是一个递归合并的过程。释放一个k级cell后，检查它所有的buddy cells，如果都free了，则合并，产生一个k+1级的cell。然后递归这个过程。
        
        这个合并过程减小了GPU的碎片化，为那些需要更高级cell的job提供了机会。

        本文能够通过离散时间片的数学归纳法证明该算法，当VC的分配是可行的时候，能够满足任何合法的cell分配请求。实际上这里的时间片就是一个逻辑时钟，每当有一个分配请求或释放请求时，时间片+1。那么就是假设在时间片i时能够满足分配请求，证明在时间片i+1时，无论集群状态如何，来到怎样的请求，还是能够满足该请求。详细证明见文章。


## Data sets & Experimental Design

<!-- 撰写实验环境的设置，实验的对象，实验的比较方面，以及实验的结果（不要列举数据，要概括谈） -->
- 实验环境
    
    96-GPU的集群，大规模的trace-driven的模拟。

- 比较方面

    - 在真实集群上，比较sharing-safety（通过queue-delay比较）
        
        分别使用YARN-CS，Gandiva，Tierasia作为调度器，分别比较Quota，私有集群，以及HiveD的queue delay，以及JCT表现。

    - 在模拟器上，使用全部的trace做实验

        - 在普通压力的集群上，比较queue-delay
        - 在高负载集群上，比较queue-delay

    - Buddy Cell Allocation算法的评估

        - 比较动态与静态绑定cell的影响。
        - 该算法有多么减小GPU碎片化
        - 算法性能

## Conclusion And Future Work
本文提出了sharing safety作为资源分配的指标，本文的方法能够在不出现over subscription的集群中，保证每个租户的训练效果、性能不比使用私有集群的效果差。但是存在一定局限性：首先需要VC的分配能够适配到物理资源中，但是当GPU资源不足时，VC的分配可能不够。而用户只能选择固定的VC。这可能会导致用户不得不为了能够运行自己的任务而选择一个比较小的VC。而随时间推移，集群有资源释放了，但是该用户的VC还是那么大，多的资源难以利用。并且本文的算法仅能针对同构集群。在异构集群上，它需要将异构集群切分成多个小的同构集群。这种做法可能会带来一些损失。
<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->
