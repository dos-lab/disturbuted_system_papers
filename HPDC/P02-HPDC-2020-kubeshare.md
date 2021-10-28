# Title

<!-- 此部分是论文标题及其引用格式，建议使用latex格式 -->
KubeShare: A Framework to Manage GPUs as First-Class and Shared Resources in Container Cloud

## Website

https://www.youtube.com/watch?v=1WQMKCGN9j4

## Citing

Ting-An Yeh, Hung-Hsin Chen, Jerry Chou: KubeShare: A Framework to Manage GPUs as First-Class and Shared Resources in Container Cloud. 173-184

## Brief introduction

<!-- 通过三五句话描述这篇文章，包括 1. 论文的应用场景；2. 论文克服已有方法的局限性；3. 论文主要的技术手段； 4. 论文的预期结果 -->
该论文主要面向机器学习的GPU分时复用场景，针对已有Round-robin方法容易导致资源碎片的问题。提出了支持Locality的GPU虚拟化和分配方法。Locality包括亲和性和非亲和性，将亲和性是将两个容器必须放在一个GPU上，反之为非亲和性。
尽量将两两GPU总资源需求为100%的容器标记为亲和性，从而达到提高GPU资源利用率的目标。

## Key Methodology

<!-- 分点写，论述论文中主要技术手段的实施过程 -->
论文的核心是如何实现GPU虚拟化，其核心思想是在Nvidia驱动后端与容器前端进行插桩，提供基于Token的GPU时间分配算法。

## Data Sets & Experimental Design

<!-- 撰写实验环境的设置，实验的对象，实验的比较方面，以及实验的结果（不要列举数据，要概括谈） -->
测试平台：4 Nvidia Tesla V100 GPU（16GB显存）；测试应用：ResNet-50，DeepLab v3应用；评价指标：GPU虚拟化隔离效果和Locality敏感的GPU分配算法的收益。实验结果显示，测试应用完成时间（性能）可以缩短1倍。


## Conclusion and Future Work

<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->
该论文面向机器学习的GPU分时复用场景，工作仍比较初级，有很大的改进的空间。
