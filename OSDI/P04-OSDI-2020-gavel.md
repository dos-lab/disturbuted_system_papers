# Title

@inproceedings{narayanan2020heterogeneity,
  title={Heterogeneity-Aware Cluster Scheduling Policies for Deep Learning Workloads},
  author={Narayanan, Deepak and Santhanam, Keshav and Kazhamiaka, Fiodar and Phanishayee, Amar and Zaharia, Matei},
  booktitle={14th $\{$USENIX$\}$ Symposium on Operating Systems Design and Implementation ($\{$OSDI$\}$ 20)},
  pages={481--498},
  year={2020}
}

## Citing

Narayanan, Deepak, et al. "Heterogeneity-Aware Cluster Scheduling Policies for Deep Learning Workloads." 14th {USENIX} Symposium on Operating Systems Design and Implementation ({OSDI} 20). 2020.

## Brief Introduction

深度学习作业运行在异构资源上，不同资源存在不同的吞吐量和运行成本，为了高效利用多种异构资源（GPU型号异构），能为公平分配、LAS，先进先出，最小化等待时间，服务质量成本最小化等调度提供异构资源感知。能够将调度最优化问题转化为异构、集中放置感知、放置位置感知的数学模型。

## Key Methodology

支持最大最小公平性、短作业优先等各种最优化调度的数学表达，具有一定的鲁棒性。
它的调度主要分为两大步骤：首先是将调度策略转化为一个或一系列的线性规划问题，解这些问题能够得到一个最优的资源时间分配矩阵。第二步，则是依照时间片轮转的方法，按照轮次来进行任务资源调度，能够在多轮后逐渐向前面得到的最优资源分配矩阵靠拢，以此来达到调度策略的目的。

1. 调度策略转化为线性规划问题

    在本系统中的每个job，定义为一个模型的训练，它拥有固定的epoch，固定的scale_factor(就是数据并行的个数)。
    
    首先定义了资源时间分配矩阵X，这个矩阵描述了在每次重新生成该分配矩阵之间的时间内，每个job在对应类型上的加速器上应该执行的时间（这里并没有考虑这种类型的加速器个数是否足够支撑scale_factor，可能存在问题）。并且定义了吞吐率矩阵T，它定义了每个job在每个类型的加速器上的吞吐率。通过X和T的计算，它定义了一个新的吞吐率的指标effective throughput。使用它来作为每个job的执行速度。

    该effective throughput是将资源时间分配矩阵X作为自变量的一个函数，那么就可以将各种调度策略（最大最小公平性、短作业优先等）使用effective throughput组成的线性规划问题来表示，其中对自变量X有各种有效性限制条件。

    对于space sharing的建模，它通过Quasar解决了space sharing的吞吐率测量问题，并且指定了最多只有两个job可以做组合。然后经过扩展资源时间分配矩阵X，在其中加入某些job的组合的行，从而统一建模问题。

    对于placement sensitivity描述的不是很清晰，需要看代码去了解。

2. 调度机制

    在给定了资源时间分配矩阵X的情况下，下面通过时间片轮转调度的方式，去尽可能接近该X目标。

    计算每个任务在过往的时间片轮次中，该任务在每种加速器上执行了多久。能够通过该时间计算出当前每个任务在每种加速器上的执行时间比例。该比例与X相比，差距最大的即为优先级最高的（更加需要分配时间片，以便与X更加接近）。

    时间片在文中使用基于经验选择的6分钟。基于时间片的策略能够避免某些占用加速器数量较多的任务的饥饿现象。

    通过这种方法，每个轮次中使用贪婪的方法选择优先级最高的任务。在每一轮使用次优的贪心决定，在后来的轮次中能够recover。当某一个job（combination）在某一轮没有执行，会在下一轮获得更高优先级；而若在这一轮运行，则它的优先级会降低，在下一轮更不容易选中。

## Data Sets

ResNet-18，ResNet-50，A3C，LSTM，Transformer，CycleGAN

## Experimental Design

在最优化目标有效表达后，提升JCT、降低等待时间，提升资源效率等成果。

## Conclusion and Future Work

从异构资源出发来衡量深度学习的性能，这些方法也能推广到其他工作负载，在同构资源下也可以有为简洁的调度目标表达方式。

基于job级别完备的数学模型，如何和算子、数据流结构结合，实现协同调度优化。

1. 不支持模型并行的建模。对于某些复杂情况，如某个模型在某个加速器上能够放入，但在某些加速器上放不下，需要模型并行。这时需要考虑的建模会更复杂。
2. 未来任务到来时，先前的资源时间分配矩阵X可能是一个次优解。
3. scale factor是数据并行的个数，由用户指定且不能修改。scale_factor作为一个影响吞吐率的指标，应该作为调度时可调整的一个可选项。固定住它的大小会限制调度算法可提升的吞吐率空间。
4. 它的数据并行scale能否支持跨GPU类型？文中说的比较模糊难以理解。
5. 对于某些数据并行的复杂情况没有予以考虑：当某个job处于如下执行状态在Gavel中是不存在的，即某个job分别在两个worker上数据并行，其中在第一个worker上独占GPU，在第二个worker上与其他job共享资源。这是因为它在对job的资源分配矩阵中，对于job combination是独立的一行，而它的scale_factor也是针对这个combination的，无法扩展到上述情况。
6. 时间片是超参数，文中测试时基于经验选择为6分钟，是否可以对它进行建模分析？针对不同workload分析不同时间片的idle job时长，对于人工指定的超参数
7. 异构仅支持GPU型号，能否支持包含CPU，IPU等其他硬件？
8. 是否可以通过对过往任务到来的情况做时序上的建模，预测未来的资源竞争的激烈程度，并针对性做出非贪婪的线性规划的解作为资源分配结果。