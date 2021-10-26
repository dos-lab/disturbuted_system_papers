# Titile

Scheduling Beyond CPUs for HPC

## Citing

Fan Y, Lan Z, Rich P, et al. Scheduling Beyond CPUs for HPC[C]//Proceedings of the 28th International Symposium on High-Performance Parallel and Distributed Computing. ACM, 2019: 97-108.

## Brief introduction

高性能计算存在对多种异构资源的调度问题（CPU和SSD），BBSched克服普通方法、资源约束方法和权重方法的不足，使用多目标约束的方法优化资源调度，其优化方法能够扩展到多维资源的情况，能够提升41%的调度性能。

## Key Methology

使用滑动窗口机制批处理任务集合，最大化节点资源使用率和突发缓存（SSD）使用率，最终计算出最优的任务放置。

## Data sets

Argonne Leader ship Computing Facility的内部数据，不开源

## Experimental Design

评价指标：节点资源使用率，SSD使用，作业等待时间，作业延迟比例。

## Conclusion And Future Work

设计的单一state of the art调度方法如果没有背景数据的支撑，很难在b以上的会议中发表。
