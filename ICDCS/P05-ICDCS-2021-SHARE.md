# Title

Communication-Efficient Federated Learning with Adaptive Parameter Freezing

## Website
https://doi.org/10.1109/ICDCS51616.2021.00012

## Citing

@INPROCEEDINGS{9546457,
author={Deng, Yongheng and Lyu, Feng and Ren, Ju and Zhang, Yongmin and Zhou, Yuezhi and Zhang, Yaoxue and Yang, Yuanyuan},
booktitle={2021 IEEE 41st International Conference on Distributed Computing Systems (ICDCS)},
title={SHARE: Shaping Data Distribution at Edge for Communication-Efficient Hierarchical Federated Learning},
year={2021},
pages={24-34}

## Brief Introduction

该论文基于分层的联邦学习，提出了最小化由云边模型聚合导致的通信成本（CCM）问题，并通过在边缘对数据分布进行调整，从而解决 CCM 问题的框架 SHARE。文章具体将 CCM 问题拆分为两个子问题，分别为最小化每轮通信成本与最小化边缘聚合数据的平均 KL 散度。

## Key Methodology

1. CCM 问题的定义

   给定一个分布计算节点集与对应的候选边聚合节点集，选择一个子集作为边缘聚合节点集，以及其相应的节点到边缘的连接关系，从而使分布计算节点到边缘聚合节点之间 *X*、边缘聚合节点与云聚合节点之间 *Y* 的通信总成本最小。
   
   而其中，计算节点到边缘聚合节点之间的成本，即为聚合节点与计算节点之间所有连接成本与聚合频率（两次聚合之间的迭代次数）与聚合期望次数的乘积。
   
   边缘聚合节点到云聚合节点之间的成本，即为两者的连接成本与聚合轮数的乘积。而 CCM 问题则是求两者和的最小值。
   
2. CCM 问题的分解

   - PCCM 问题，以 *X* 与 *Y* 为自变量，最小化每轮聚合的通信成本，可以理解为目标函数中不计算总聚合轮数的 CCM 问题。

   - KMM 问题，最小化聚合轮数，即以 *X* 与 *Y* 为自变量，最小化边缘聚合节点数据真实分布与均匀分布之间的平均 KL 散度，

   - DD-CCM 问题，即前两者的 tradeoff。

3. 算法设计

   - 分布计算节点的连接

     对于给定的边缘聚合节点，为每个计算节点找到其对应的聚合节点，以满足 DD-CCM。在算法中，每个计算节点对所有聚合节点进行遍历，找出 DD-CCM 值最小的聚合节点作为自己的聚合节点。
     
   - 边缘聚合节点的选择
   
     随机选择候选节点中的一部分节点作为初始方案，遍历所有方案外的节点，如果符合 DD-CCM 则加入到方案中。遍历结束后，遍历所有方案内的节点，如果不符合 DD-CMM，则排除到方案外。然后对于每个方案外的节点，遍历所有方案内节点，如果加入前者并排除后者符合 DD-CMM，则执行操作。


## Data sets & Experimental Design

实验数据：MNIST with standard CNN, CIFAR-10 with ResNet-18

1. 

实验对象：基于云的联邦学习（无边缘）

实验结果：SHARE 的通信成本低了约90%。

2. 

实验对象：只考虑通信成本的 CPLEX（CC）、只考虑数据分布的贪心算法（DG）

实验结果：在相同的聚合轮数下，SHARE 的模型准确率更高，在相同的通信成本水平下，SHARE 的模型准确率更高。


## Conclusion And Future Work

该论文着重于解决分层联邦学习场景下的最小通信成本问题。但是本文在进行计算节点的取舍时，只考虑了通信成本，没有考虑每个节点对模型的准确率贡献，或者说“学习质量”，因此并不是最优方案。
