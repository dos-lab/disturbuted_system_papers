# Cited Title

@inproceedings{10.1145/3419111.3421279,
author = {Shi, Shouqian and Yu, Ye and Xie, Minghao and Li, Xin and Li, Xiaozhou and Zhang, Ying and Qian, Chen},
title = {Concury: A Fast and Light-Weight Software Cloud Load Balancer},
year = {2020},
booktitle = {Proceedings of the 11th ACM Symposium on Cloud Computing},
pages = {179–192},
numpages = {14},
keywords = {data center, mobile edge computing, load balancing, software load balancer, cloud computing},
series = {SoCC '20}
}


## Brief introduction

在云服务中，现有的负载均衡算法使用摘要存储包状态，不仅消耗大量存储资源，而且摘要冲突会破坏包的一致性，且由于每次新建连接都会刷新数据层，过高的刷新频率影响了吞吐量与包的一致性。论文提出了名为 Concury 的负载均衡算法，使用基于 Bloomier filter 的 OthelloMap 代替普通的哈希表与基于均衡随机的加权负载均衡，降低了数据层的刷新频率，减少了内存占用，并且避免了错误命中的发生。

## Key Methology

1、在数据层上，使用以单个VIP为单位的、使用 Bloomier filter，存储从 state 到 DIP 的映射：使用两个数组作为存储部分的数据结构，两个数组元素的异或结果为DIP对应的下标，避免直接存储 state 或者是 state 的摘要。

2、加权负载均衡。根据权值为DIP分配大小不同的Dcode区间。由于使用 Bloomier filter，所选择的两个数组元素中只要有一个为空，则表示该Dcode为空闲状态。

3、在控制层上，使用 OthelloMap 存储 VIP 数组与记录当前状态集合的 BAS 表并对 state-DIP 数值对进行增删改查。

4、反应控制与数据层的更新。只有当VIP对应的DIP池发生改变时，需要进行数据层的更新（只有此时可能会影响包的一致性），且只更新与修改相关的 VIP 的相关数组，并限制更新大小为1024 bits。

5、通过 Dcode 与 DIP 的映射，在 DIP 发生变化时修改 Dcode，保持状态与其对应的 DIP 不变，从而实现网络动态变化中包的一致性。


## Data sets & Experimental Design

1、算法评估

实验环境：Intel i7-6700 CPU, 3.4GHZ, 8 MB L3 Cache shared by 8 logical cores, 16 GB memory (2133MHz DDR4)

实验对象：Concury、Maglev、SilkRoad

系统结构：人工设置大小两种规模的DIP-E、DIP-V

实验数据：Facebook数据中心网络的真实通信数据（没有数据集名称）以及由LFSR随机生成状态的包

比较方面：存储消耗、吞吐量、控制层的响应时间与更新可扩展性。

2、真实网络环境中的应用评估

实验环境：1）a lab 100GbE built by Mellanox MCX516A-CDAT NICs

​				   2）CloudLab

比较方面：包一致性、吞吐量

结果：100%的包一致性，且达到了较高的吞吐量（声称其单线程的效果是现有的软件LB中最好的）


## Conclusion And Future Work

本文设计了一种应用于云边数据中心的、基于软件的负载均衡方案。文中提出的用于存储 state-to-DIP 的基于 Bloomier filter 的数据结构很妙，实验也证实了其存储性能、负载均衡、维持包一致性等方面的优越性。文章的下一步工作是将 Concury 推广到手机客户端环境中。
