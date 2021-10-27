# Titile

Tiresias: A GPU Cluster Manager for Distributed Deep Learning

## Citing

Gu J, Chowdhury M, Shin K G, et al. Tiresias: A {GPU} Cluster Manager for Distributed Deep Learning[C]//16th {USENIX} Symposium on Networked Systems Design and Implementation ({NSDI} 19). 2019: 485-500.

## Brief introduction

针对DL任务的调度问题，已有工作存在较多问题，kubernetes、yarn的静态调度器过于简单，无法满足实际需要；Optimus等基于DL的建模，其准确度不高（收敛曲线不够平滑）；基于约束和数据本地性考虑，以最少机器和最少网络通信的方法也未必能够提升性能。Tiresias为了有效的调度GPU，最小化JCT，同时提升efficiency，通过对身产环境的job信息进行分析，面向不同的DL模型，提出两种基本思想：基于离散的2D-LAS和2DGittins-index方法。

## Key Methology

同时考虑空间约束和时间的不确定性，使用Gittins和LAS两个优先级设置方法，使用多级队列（不同优先级）处理避免饥饿等情况。

## Data sets

wordload没有开源的数据集，
cnn模型选择TensorFlow官方的模型（https://github.com/tensorflow/benchmarks）

## Experimental Design

1. 基准测试：yarn
2. 衡量指标，JCT的缩小比例
3. 实验环境：集群环境与模拟统计的工作负载（480个作业的训练时间和资源使用量）

## Conclusion And Future Work

在JCT这一个指标上有显著提升，其他指标也许无法保障，对DL的粒度控制的也相对较粗。
