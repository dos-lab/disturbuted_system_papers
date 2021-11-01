# Cited Title

@inproceedings{10.1145/3419111.3421299,
author = {Chen, Chen and Weng, Qizhen and Wang, Wei and Li, Baochun and Li, Bo},
title = {Semi-Dynamic Load Balancing: Efficient Distributed Learning in Non-Dedicated Environments},
year = {2020},
booktitle = {Proceedings of the 11th ACM Symposium on Cloud Computing},
pages = {431–446},
numpages = {16},
keywords = {load balancing, synchronization, distributed learning},
series = {SoCC '20}
}


## Brief introduction

在基于 BSP 的无专用节点机器学习过程中，当 worker 的性能不足以匹配负载时，会变成 deterministic straggler，拖累整体的训练效率。

该论文在静态与动态负载均衡两种方法的基础上提出了半动态负载均衡方案（在每轮迭代内 worker 的负载是静态的，在不同的迭代之间负载是动态的），克服了静态负载均衡面对非专用集群的低灵活性与动态负载集群的低可用性。

该论文通过最近的若干次迭代来推测未来短期的执行状态，并利用BSP在每轮迭代结束时的同步路障来获取全局workers 的状态，并通过调整每个 worker  的 batch size 来达到更优的负载均衡的目的。

该方案可以有效减少 straggler，提升训练效率。文中GPU集群效率提升了54%，CPU 集群效率提升了38.7%。

## Key Methology

1、对于 CPU 集群：由于处理时间与 batch size 成线性关系，因此需要求其系数，即瞬时的处理速度。使用 NARX 模型，一种 RNN 的扩展，输入序列为采样处理速度的历史值、CPU 以及内存的历史与当前使用率，以建立从输入序列到预测速度的非线性函数F，进而调整 batch size。

2、对于 GPU 集群，获取监视窗（为了减少非确定的随机变化带来的扰动）内一个耗时最小且内存充裕的 leader 和一个耗时最长的 straggler，分别增减两者的batch size，当出现振动时，通过减小步幅并增大窗口尺寸来进行微调。

3、在梯度聚合时，使用以 batch size 为权值的加权平均代替BSP的一般平均，以避免不同的 batch size 对梯度的影响。


## Data sets & Experimental Design

两个实验对象：多样 GPU 集群 & 多样 CPU 集群

1、GPU集群：
模型与数据集： CifarNet & ResNet-32 on CIFAR-10，Inception-V3 on ImageNet
使用单次梯度更新的时间衡量硬件有效性，使用到达目标准确率所需的更新次数来衡量统计有效性，到达目标准确率所需的总时间衡量总体的有效性。
并使用四个GPU组成的微观集群对 batch size 与处理时间的变化进行了验证。

2、CPU集群：
模型与数据集：SVM on a malicious URL，ResNet-32 on CIFAR-10 dataset
使用平均迭代时间衡量硬件有效性。
将NARX模型与其他模型进行了对比测试。
测试了算法中存在的阻塞对CPU集群的影响。


## Conclusion And Future Work

本质都是根据性能为不同的 worker 分配不同大小的 batch。动态在于在迭代开始时调整 batch 的大小，静态在于在迭代内 batch 的大小是不改变的。并针对 GPU 集群与 CPU 集群的不同特点给出了相应的调整 batch 的方案。
