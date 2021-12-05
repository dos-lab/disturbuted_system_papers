# Title

Replica-Group Leadership Change as a Performance Enhancing Mechanism in NoSQL Data Stores

## Website
https://doi.org/10.1109/ICDCS.2018.00147

## Citing

@INPROCEEDINGS{8416410,
author={Papaioannou, Antonis and Magoutis, Kostas},
booktitle={2018 IEEE 38th International Conference on Distributed Computing Systems (ICDCS)},   title={Replica-Group Leadership Change as a Performance Enhancing Mechanism in NoSQL Data Stores},
year={2018},
pages={1448-1453}

## Brief Introduction

该论文提出了一种应用于 NoSQL 的组复制自动重构方案，即在主节点将要执行如 LSM-tree compaction 或数据备份等资源密集型后台任务前，自动对主节点进行变更。该论文使用 MongoRocks 进行实现，经实验，系统吞吐率最高可以提升23%到35%。

## Key Methodology

1. 方案设计

   - 重构时机

     当资源密集型后台任务（本文只讨论了 LSM-tree compaction 和数据备份这两种情况，接下来的所有讨论都只有这两种情况）即将在主节点上运行时，其对性能的影响大于执行重构成本（系统暂时不可用）时，则进行复制组的重构。

   - 主节点选择策略
   
     采用 replica-ranking （RR）算法，以所有副本的 gossip 状态作为选择依据，即没有正在执行后台任务的次节点考虑为主节点的候选。当存在多个候选时，有两种选择方案：
   
     - 最近完成优先（MRC），即选择最近完成内部后台任务的节点。由于内部任务一般具有周期性，因此最近完成内部后台任务的节点会有一个较长的无后台任务期望。
     - 最长主节点优先（LSP）。所有的副本都会在内存中留下读缓存，但是只有主节点会使用它，次结点则会无视它，因此 LSP 的内存访问命中率更高。
     - 也存在两种方案结合的情况：当最近的原主节点完成后台任务后立即开始重构，重新将其设定为主节点（LSP 的变种 preferred primary，姑且称之为 PP），且只选择一个节点作为长期的主节点（应该是考虑到了反复执行的情况）。
   
     若没有合适候选，则主节点会推迟将要进行的后台任务，并在收到其他次节点状态更新时进行重新评估。当主节点认为重构代价过大（没有说衡量标准）或者延迟时间过长时，会开始执行后台任务。
   
2. 具体实现

   MongoDB 内置对组复制的重构支持。每个复制组的成员都有优先级属性，该属性与其选举超时阈值成正比，超过阈值则会在符合条件的前提下提议开始进行选举。在该论文的实现中与 RR 算法结合，将运行在所有复制组上的后台任务（全局视角）作为算法输入，所有成员在 MongoDB 的每次心跳时报告自身状态，并且使用 RocksDB 的自带 API 以暴露正在运行的 compaction 计数。RR 算法只选取不在执行 compaction 的节点作为候选，并使用 PP 策略：主节点监视前任的状态，并在其完成内部任务时触发选举，使前任重新成为主节点。

   RocksDB 告知 MongoDB 关于 compaction 任务的到来并发送执行请求，如果接受的复制组为次节点则开始执行，反之将其置入挂起队列中进行等待。当目标节点变为次节点时，再从挂起队列中将其调出。

## Data sets & Experimental Design

实验环境：MongoDB 3.7 & RocksDB 5.7

实验数据： Yahoo Cloud Serving Benchmark v0.11（YCSB）

比较方面：吞吐率

实验结果：标准的MongoRocks 系统会在主节点执行 compaction 或数据备份任务的时候出现吞吐率降低的情况，而有自动重构机制的 MongoRocks 则更稳定，且平均吞吐率更高。




## Conclusion And Future Work

提出了一种根据任务请求转移主节点的机制。根据请求来源分为了外部请求（数据备份）与内部请求（compaction），但是讨论的范围可以再大一些，而且对于主节点的转换成本没有微观的表示，比较起来有些抽象。
