# Title

Elasecutor: Elastic Executor Scheduling in Data Analytics Systems

## Citing

Liu, Libin, and Hong Xu. "Elasecutor: Elastic Executor Scheduling in Data Analytics Systems." Proceedings of the ACM Symposium on Cloud Computing. ACM, 2018.
## Brief Introduction
Executor是spark、storm中的概念，其上可以运行多个task，executor可以看做是资源分配的基本单位（而不是task），当前executor的静态资源分配方式不能满足实际的需要，导致资源利用率比较低。需要能够动态调整executor的资源配额，提升资源的利用率。

## Key Methodology

（1）Benchmark：HiBench中较为经典的、具有一定代表性的spark作业，一个三个数据集，八类算法；
（2）统计时间线上各个benchmark的资源变化情况，不平衡，使用率低是主要问题；
（3）如何能使makespan的时间最少是核心优化问题，公式四；
（4）使用改进的一维装箱算法，每次优先选择后续时间线中遗留资源最大的机器（Dominant Remain Resource，看做是DRF的变种），以DRR作为启发式规则，考虑时间序列的资源变化情况；
（5）没有考虑异构资源、放置约束等情况
（6）每个benchmark使用的executor数量等其他信息采用默认配置；

## Data Sets


## Experimental Design


## Conclusion and Future Work

改进的一维装箱算法是核心贡献。论文中提出三处改进：支持放置约束；支持基于deadline的服务质量保障；支持与yarn等调度器的集成
其使用场景具有明显的局限性，benchmark不能反映真实生产环境的需求，其固定的负载变化扩展性未必好，也不支持长任务；其是基于spark实现，只能限制在具有executor概念的框架，可扩展性不好，与其他framework、task（container）等类型的调度如何进行集成，如何分辨不同层次的干扰是核心问题。