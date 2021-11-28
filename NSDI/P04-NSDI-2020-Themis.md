# Title

<!-- 此部分是论文标题-->
Themis: Fair and Efficient GPU Cluster Scheduling

## Website
<!-- 网址，有DOI的建议用DOI地址-->
https://www.usenix.org/system/files/nsdi20-paper-mahajan.pdf

## Citing

<!-- 引用格式，建议使用latex格式-->
@inproceedings{mahajan2020themis,
  title={Themis: Fair and efficient $\{$GPU$\}$ cluster scheduling},
  author={Mahajan, Kshiteej and Balasubramanian, Arjun and Singhvi, Arjun and Venkataraman, Shivaram and Akella, Aditya and Phanishayee, Amar and Chawla, Shuchi},
  booktitle={17th $\{$USENIX$\}$ Symposium on Networked Systems Design and Implementation ($\{$NSDI$\}$ 20)},
  pages={289--304},
  year={2020}
}

## Brief Introduction

<!-- 通过三五句话描述这篇文章，包括 1. 论文的应用场景；2. 论文克服已有方法的局限性；3. 论文主要的技术手段； 4. 论文的预期结果 -->
本文关注深度学习训练任务的调度问题，它主要关注调度公平性这一指标。先前的针对公平性的调度方案大多数仅针对大数据任务，其没有关注到深度学习任务的长时间，放置敏感性的特征，先前的方法容易造成sharing incentive, Pareto efﬁciency, envy-freedom这三大公平性性质的违反。本文给出了一个新的针对DLT任务的公平性指标finish-time fairness，并给出一套基于拍卖和租约的调度算法，能够实现PE，EF，SP的性质，并尽可能保障SI性质的达成。最后在Yarn上实现了双层调度器实现了该调度算法。最终能够有效提升用户的公平性。

## Key Methodology

<!-- 分点写，论述论文中主要技术手段的实施过程 -->
本文首先定义了app。app是每个用户提交的一个或一组DLT job，以app作为基础单位实现公平性。其中可以理解app为一个用户提交的一组超参数调优job。而每个job可以是分布式训练的，它可以包括多个并行的task（组成一个数据并行的训练）。

1. 公平性在DLT任务上问题

    1. DLT任务的长度

        与大多数大数据任务相比，DLT任务都很长，在DRF中（先前的基于大数据的公平性调度方案），一旦资源可用时，就将那些最不公平的app的task放置到上面。这对于大数据应用有效，因为任务完成较快，有很多机会进行资源重新分配以便达到公平性。但是ML任务太长，可能导致排队等待的任务长时间得不到运行，这会导致SI违反。

        Tiresias的LAS能够避免饥饿，但是违反其他的性质，因为没有考虑placement-sensitivity。

    2. placement-sensitivity

        文中分析VGG16和Inception-v3。VGG16更需要consolidation。但是还是我之前想的：每个模型的需求带宽不同。
        
        而带宽足够时，consolidation和分散开的性能不会差距很大。

        文中给出了具体的两个例子，分别说明了不考虑placement-sensitivity的方案会如何违反公平性指标SI、PE、EF。

2. 新的公平性指标 finish-time fairness

    Finish-time fairness: p定义为$T_{sh}/T_{id}$。

    $T_{sh}$为这个app在共享集群中的结束时间。它包含了由于placement和排队延迟造成的速度减缓。

    $T_{id}$则为这个app独占1/N的集群的结束时间。

    它能够反映app在集群内部受placement的影响。并且能够反映长期执行的任务的公平性。

    那么SI目标就可以用p <= 1表达。

    但是该p值需要在不同GPU分配(placement)下的测量。然而预测不同app在不同分配下的p很困难。

    所以，本文通过允许app自己表达它对各种分配方式的偏好程度。实际上，app将提供一个函数，即$p(.)$，它的自变量是各种类型的GPU allocation，将会返回一个在这种GPU allocation下的p值。

3. 调度机制：Partial Allocation Auctions（部分分配的拍卖）

    - one-shot auction

        如果简单地按照p的值排序，最先选择p最小的开始执行。这个方式的问题是：用户给出的p值可能是假的，这样贪婪的选择会让那些造假的用户更有可能获得更多资源。

        Auction（拍卖）是为了解决以上问题而提出的。

        使用partial allocation auction（这是一个已有的方法，并不是新提出的）。
    
        这个方法被验证于：
        
        1. 激励用户告知真实数据。
        2. 对于不可分割货物的子集的数学建模非常合适。

        该partial allocation的拍卖方法具体比较复杂，这里简单描述：
        1. 首先，通过计算，能够得到为每个app的一个资源分配，使得每个app若要提升性能必须牺牲其他app（PE性质）。该值定义为pf。
        2. 随后，为每个app i计算一个，若该app不存在时，的pf值，定义为$pf^{-i}$。
        3. 随后，计算为每个app i计算一个系数$c_i$，它通过上面的两个pf值经过计算再相比得到。它能够反映的意义是：这个ci反映了app i的存在对集群内其他app的性能造成了多大的下降。
        4. 随后，将第一步算出的pf分配乘以这个ci系数，得到一个最终的分配结果。注意：这个分配结果是不完全的，会剩余一些资源，所以算法是partial的allocation。

        该算法中的$c_i$系数的提出是避免用户通过说谎而受益的关键。在先前的论文中已经得出结论，它的存在能够激励app说真话（告知真实的p值）。$1-c_i$可以理解为隐藏的payment。

        然而，由于没有将全部的资源都分配出去，所以会无法满足SI性质。从直觉上来说，partial allocation不满足SI是因为剩余了部分GPU没有分配。

    - round-by-round的拍卖机制

        所以提出了多轮auction，能够解决partial allocation auction的SI满足性问题。

        多轮auction中，将GPU分配的结果只与一个租约长度的时间结合。
    
        当租约结束后，释放的GPU被重新拍卖。

        这样还能能够解决在线上时，任意的一个资源可用事件（app失败，到达，cluster配置更改等）。

        每轮auction中，根据一个f参数，首先挑选出那些p值最大的那1-f比例的app，p值越大则表示越有可能违反SI。f是整个系统的一个超参数。

        通过f比例缩小需要考虑的app数量，能够减小SI违反的概率，并且减小资源竞争，从而减少hidden payments。

        经过多轮的拍卖，这种过滤能够最大化满足SI的app数量。
        
        考虑一个不公平的app输掉一次拍卖，它在将来的拍卖中有更高概率获胜。

        这是因为，胜利的app，随着它的执行，它剩余的时间变短，那么在将来评估新的p值时，它会比上次输掉的app更具有劣势，所以在将来会有可能脱离1-f的比例。

        f越接近1时，则每次挑选的p值最大的任务数越少。因为每次都是只有一个资源可用，再执行拍卖，那么在这时，参与拍卖的p值最大的任务很少，它们更有可能获得GPU分配，所以f越接近1时，越容易达成SI。而f越接近0时，参与拍卖的越多，由于在拍卖时我们考虑到了更多的app，更多的p值和GPU分配的可能性，这会让我们更容易找到placement更优的app组合，所以这种情况下更容易获得更好的性能。这里存在trade-off。

4. 双层调度器

    - 先前的问题

        悲观的调度器，限制了单个app的可见性，因为可用资源被调度器分隔开，每次只向一个app展示其中一个。

        乐观的调度器在所有的app中都共享了状态，所有app都能看到全部的资源。所有app竞争资源以及资源的分配决定是由多个app使用事务同时共同决定的。这种lock-free的方式让本文的finish-time fairness的这种global的policy很难实现。

        所以需要引入一个半乐观的双层调度器。

    - 工作流

        每个GPU都具有一个租约lease。当它的租约到期时，则触发一个资源可用的事件：

        工作流：
        
        当有资源可用时：
        
        阶段一：App可见
        1. ARBITER询问所有app的finish-time fairness指标
        2. ARBITER开启一次拍卖，将所有可用资源数告知给每个app。
        3. 每个app投标（给出它想要的资源分配）

        阶段二：分配
        1. 获得全部app的投标后，执行算法获得调度分配。
        2. 分配结果传输给每个app的agent，将分配传达给app。

    - 好处

        半乐观的并发控制：

        1. 和全乐观相同的地方：每个app能够并发地对全部资源可见。

        2. 和悲观相同的地方：资源分配是不存在冲突的，因为资源调度器做的调度决定是不存在冲突的。




## Data sets & Experimental Design

<!-- 撰写实验环境的设置，实验的对象，实验的比较方面，以及实验的结果（不要列举数据，要概括谈） -->

- 实验环境：

    64GPU集群，GPU类型都一样，但是placement不完全一样。部分是2GPU的机器，部分是4GPU的机器。

    模拟器：模拟了4种GPU放置的256GPU的集群。

- 比较对象

    SLAQ、Gandiva、Tiresias、SRTF、Optimus、SRSF

- 基于真实集群的实验

    - 比较finish-time fairness指标
    - 比较总利用的GPU时间。该值越短则说明利用效率越高。
    - 比较GPU分配的碎片
    - 高竞争下的finish-time fairness。
    - 调度器的overheads

- 基于模拟器的实验

    - 将使用network的激烈程度的app按比例上升，比较最大的finish-time fairness的值，以及GPU时间分布。
    - 比较app投标的误差率。
    - 说谎的app的性能比说真话的app的性能差。
    - 系统对f参数和lease时间参数的敏感性

## Conclusion And Future Work

<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->
提出了一个新的长期的公平目标finish-time fairness。然后提出了一个两层半乐观调度架构，DLT app可以在其中对拍卖中提供的资源进行投标。