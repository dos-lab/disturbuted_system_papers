# Title

Boki: Stateful Serverless Computing with Shared Logs

## Website
https://doi.org/10.1145/3477132.3483541

## Citing

@inproceedings{10.1145/3477132.3483541,
author = {Jia, Zhipeng and Witchel, Emmett},
title = {Boki: Stateful Serverless Computing with Shared Logs},
year = {2021},
pages = {691–707},
numpages = {17},
keywords = {consistency, shared log, Serverless computing, function-as-a-service},
series = {SOSP '21}
}

## Brief Introduction

该文章提出了一种应用于有状态无服务器计算的 serverless runtime，Boki。其通过使用分布式共享日志进行状态管理，以满足系统的弹性、数据本地性以及资源效率的需求。Boki 使用元日志（metalog）统一解决日志记录的全局排序、数据一致性与容错三个问题。经验证，共享日志可以将无服务器负载加速最高达4.7倍。

## Key Methodology

1. 结构

   - 存储节点

     Boki 将物理日志分散存储，每个日志被分散存储在若干个存储节点上，而一个节点可能存储着包含属于不同日志的日志片。

   - 排序结点（sequencer node）
   
     该节点运行 sequencer ，使用 primary-driven protocol （类似日志的主从复制，只有 primary sequencer 能更新元日志，每次更新，primary sequencer 都会把元日志的副本发送给所有 secondary sequencer 并对更新进行投票）来对原日志进行存储与更新。
   
   - LogBook 引擎
   
     该进程运行在函数节点上，存储物理日志索引，并根据元日志对其进行维护。以及缓存日志记录。
   
   - 控制层
   
     使用 ZooKeeper 存储相关设置，包括每份物理日志的存储、sequencer、索引；网关、函数、存储与 sequencer 的地址、从 LogBook 到物理日志的一致性哈希参数等。当发现失效节点时，对其进行重设定。
   
2. 日志添加的工作流程

   Boki 的物理日志分散为若干碎片（shard），每个碎片又分成若干个副本，每个副本存储在一个存储节点上，而每个函数节点控制一个碎片。对于每个函数节点，其 LogBook 引擎都包含着计数其记录数的计数器。添加新日志时，就将计数器的新值作为当前记录的计数值，然后将新的记录复制给所有负责备份其日志碎片的存储节点中，由此，对于一份包含有 *M* 个碎片的日志及其上的记录，可以表示为长度为 *M* ，以计数值为索引的向量 *v*。通过以元素为单位比较不同存储节点的 *v*的最小值，可以得到全局向量（即被完整复制过的日志记录）。Primary sequencer 周期性地将全局向量代表的日志记录添加到元日志里。

3. 物理日志的虚拟化
   
   Boki 将物理日志分散存储为多个 LogBook，每个 LogBook 分成可变数量的碎片，在读取 LogBook 时，负责相应碎片的函数节点通过日志索引来读取对应的记录。Boki 只在数据读取时进行一致性检查，而元日志按顺序记录了日志索引的版本，因此函数在读取元日志位置为 *l* 的日志记录时，无法选择位置在 *l* 之前的日志索引版本。相应地，在 *l* 进行写操作的函数，后续再进行读操作时无法选择 *l* 之前的位置，保证了单调读与读己所写的一致性。
   
4. 重构协议
   
   当节点失效时，使用 Delo 的 log sealing protocol（日志封锁协议？）对所有当前的元日志进行封锁，即 primary sequencer 不再执行日志添加操作，且 secondary sequencer 不再接受从 primary sequencer 发来的元日志的 entry。所有元日志进行封锁后，Boki 执行重构，之后物理日志将从新的空元日志开始记录。


## Data sets & Experimental Design

1. workflow 测试

   实验数据：Beldi workflow workloads

   实验对象：Beldi（基于Nightcore）

   比较方面：时延（中位数）

   实验结果：从20-700（次 / 每秒）吞吐量范围内，Boki 的时延中位数均低于 Beldi。

2. 持久的对象式存储

   实验数据：Retwis workload

   实验对象：MongoDB

   比较方面：吞吐量、时延

   实验结果：在不同客户端数量下，Boki 的吞吐量均高于 MongoDB，在处理事务方法调用时 Boki 时延较低，处理非事务方法调用时，MongoDB 时延较低。

3. 无服务器信息队列

   实验对象：Amazon Simple Queue Service（SQS）、Apache Pulsar

   比较方面：吞吐量、传输时延

   实验结果：在不同数量比的生产者与消费者设置里，Boki 的吞吐量始终最高，且传输时延最低。

   


## Conclusion And Future Work

本文提出了首个使用分布式共享日志对有状态的无服务器计算进行状态管理的系统 Boki，并使用元日志作为解决共享日志系统问题的统一方案（日志排序、读一致性、容错），并建立了相关的兼容库。
