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

深度学习作业运行在异构资源上，不同资源存在不同的吞吐量和运行成本，为了高效利用多种异构资源（GPU型号异构），能为公平分配、LAS，先进先出，最小化等待时间，服务质量成本最小化等调度提供异构资源感知。能够将调度最优化问题转化为异构、集中放置感知的数学模型。

## Key Methodology

支持最大最小公平性、短作业优先等各种最优化调度的数学表达，具有一定的鲁棒性。

## Data Sets

ResNet-18，ResNet-50，A3C，LSTM，Transformer，CycleGAN

## Experimental Design

在最优化目标有效表达后，提升JCT、降低等待时间，提升资源效率等成果。

## Conclusion and Future Work

从异构资源出发来衡量深度学习的性能，这些方法也能推广到其他工作负载，在同构资源下也可以有为简洁的调度目标表达方式。

基于job级别完备的数学模型，如何和算子、数据流结构结合，实现协同调度优化。
