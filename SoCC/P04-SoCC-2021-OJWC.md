# Cited Title

@inproceedings{10.1145/3472883.3487007,
author = {Ambati, Pradeep and Bashir, Noman and Irwin, David and Shenoy, Prashant},
title = {Good Things Come to Those Who Wait: Optimizing Job Waiting in the Cloud},
year = {2021},
booktitle = {Proceedings of the ACM Symposium on Cloud Computing},
pages = {229–242},
numpages = {14},
keywords = {cost-efficiency, Cloud computing, job scheduling},
series = {SoCC '21}
}


## Brief introduction

本文是对云计算场景下两种等待策略，即 Long Jobs Wait (LJW) 和 Short Waits Wait (SWW) 的优化。传统的等待策略使用机器学习的方法对任务的运行时间进行预测。一般情况下，短任务占了总任务的绝大部分，但长任务占了总计算时间的绝大部分，使用机器学习对任务运行时间进行预测准确率很低。本文使用推测执行对 LJW 进行优化，而对于 SWW，使用预测等待时间来代替预测任务的运行时间，从而优化云计算集群中任务的等待时间与支出成本。

## Key Methology

1、对 LJW 的优化：

​	LJW 原理：使用机器学习的方法预测任务预计执行时间，如果大于某个阈值 t，则会强制等待使用低成本的固定资源（fixed resources）运行，而短任务则不会等待，视情况在固定资源或高成本的按需资源（on-demand resources）上运行。特点是服务器成本较低。

​	优化：该论文不使用机器学习对任务的预计执行时间进行预测，而是采用推测执行的方式：先来先服务，如果当前没有可用的固定资源，接下来到达的所有任务都将在按需资源上运行较短的时间 t，使短任务能在该时间内完成运行。而对于没有运行完成的长任务，将其杀死并使其等待空闲的固定资源。而由于短任务占了总任务数量的绝大部分，因此重启长任务的额外开销有限，可以视为另一种“准确预测任务运行时间”的成本。此外，使用spot VM（可被抢占的 VM）代替按需资源，可以在任务等待时间基本不变的情况下，进一步降低服务器成本。

2、对 SWW 的优化：

​	SWW 原理：使用机器学习的方法预测任务预计执行时间，如果任务的预计等待时间小于阈值 b，则令其等待空闲的固定资源，否则立即在按需资源上运行。特点是任务的等待时间较短。

​	优化：使用机器学习（文中为随机森林）的方法，以集群状态参数为特征，预测任务的预计等待时间。模型预测执行时间是以单个任务为尺度，而预测等待时间是以多个（所有正在运行中的）任务为尺度，根据大数定律，预测任务的等待时间的准确率要显著高于预测任务的执行时间。


## Data sets & Experimental Design

不同模型之间的对比：

实验环境：m5.16xlarge VMs (fixed resources)
实验数据：未公开数据（a year-long batch workload consisting of 14 million jobs run on a 14.3k-core cluster）、Google trace
实验对象：完美预测的 LJW 与 SWW 、传统的基于机器学习模型的 LJW 与 SWW、优化的 LJW 与 SWW
比较方面：on-demand cost & mean wait time

不同规模 fixed resources 的对比：

实验环境：100~225 m5.16xlarge VMs
实验数据：同上
实验对象：完美预测的 LJW 与 SWW 、优化的 LJW 与 SWW
比较方面：total cost increase、mean wait time


## Conclusion And Future Work

本文的模型有很多限制，可以考虑从这些限制入手进行改进：

1、模型假设所有的任务的优先级一致，但是并不一定符合实际情况。

2、在实际中，调度者往往还需要考虑到任务对资源类型的限制要求、DDL、资源需求（内存、核数等）的可变性、并行度等。
