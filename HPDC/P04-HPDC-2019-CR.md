# Title

Adaptive Resource Views for Containers

## Citing

Huang H, Rao J, Wu S, et al. Adaptive Resource Views for Containers[C]//Proceedings of the 28th International Symposium on High-Performance Parallel and Distributed Computing. ACM, 2019: 243-254.

## Brief Introduction

资源的不正确分配会影响到容器、虚拟机的正常性能，导致资源竞争、资源空闲和资源弹性程度差等问题。在容器内部资源视图中，虽然可以通过cgroup来限制资源使用，但容器应用仍然可以看到整个物理机的资源情况，从而错误估计实际可用的资源。为了解决这个问题，容器资源视图通过sys_namespace机制提供给每个容器独立的可用CPU、RAM容量，提供Sysfs接口让容器查询可用容量，可用容量的配置通过两个改进的爬山算法自适应获得到当前的可用容量；其实现是基于linux内核的系统事件机制。基于JVM的单机应用和大数据应用验证了其相较于静态资源分配、基于配置的动态分配的提升效果。

## Key Methodology

基于容器虚拟化应用放入资源可用性和约束之间的矛盾，在java应用中体现的最为明显，sys_namespace用于计算可用资源，Virtual Sysfs提供容器内的查询接口，Ns_Monitor截获Linux的系统事件来调整容器对应cgroups分配，通过jvm上的典型案例验证容器资源视图的重要性。

## Data Sets
均是开源数据集Twitter，Criteo，MovieLens 聚焦三类主要算法：LDA，PageRank等

## Experimental Design

Centos 7 单机版系统，运行基于JVM的性能基准测试：
Dacapo, SSPECjvm2008, Hibench

## Conclusion and Future Work

1. 性能分析扩展：影响程序执行的要素包括物理机，虚拟机，虚拟机中的程序环境配置，应用程序较多层次，JVM应用外C++、C和少量的python也需要对应的性能调整方案，且文中没有分析；
2. 场景扩展：新型应用对资源需求使用的干扰，重新改造文中所述的具体可用资源分配算法；
3. 结合新的硬件：除CPU和内容容量外，能否自适应更多的资源(集成显卡、独立显卡、IO等)

