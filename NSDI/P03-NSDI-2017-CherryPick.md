# Title

<!-- 此部分是论文标题及其引用格式，建议使用latex格式 -->
CherryPick: Adaptively Unearthing the Best Cloud Configurations for Big Data Analytics

## Website

https://www.usenix.org/conference/nsdi17/technical-sessions/presentation/alipourfard

## Citing

Alipourfard O, Liu H H, Chen J, et al. Cherrypick: Adaptively unearthing the best cloud configurations for big data analytics[C]//14th {USENIX} Symposium on Networked Systems Design and Implementation ({NSDI} 17). 2017: 469-482.

## Brief introduction

<!-- 通过三五句话描述这篇文章，包括 1. 论文的应用场景；2. 论文克服已有方法的局限性；3. 论文主要的技术手段； 4. 论文的预期结果 -->
该论文面向周期性大数据应用在公有云计算环境中运行时的云配置优化问题，开创性地采用贝叶斯优化（Bayesian Optimization）方法搜索最优云配置（云配置指云主机类型、云主机个数）。论文提出当前云配置优化系统大多基于性能建模（例如Ernest@NSDI 2016）方法，难以兼顾模型构建的准确性、开销与适用性。为此，该论文基于概率理论将云配置优化问题转化为黑盒搜索问题，通过周期性大数据应用自身迭代过程持续搜索优化的云配置，直到达到预期的优化效果（公式 1：保障应用执行时间的同时尽可能减小云配置开销）。

## Key Methology

<!-- 分点写，论述论文中主要技术手段的实施过程 -->
该论文的贡献在于开创性地采用概率搜索方法来解决云配置优化问题，实现的贝叶斯优化方法原理直观有效（Figure 5），相比于当时主流的性能建模方法优势在于能够有效解决建模开销大、适用性差的问题。CherryPick概率搜索方法的局限性在于仅适用于周期性大数据应用。后续国内外相关研究方向有很多论文都借鉴、改进了CherryPick的实现，例如：HeterBO@IPDPS 2020、Arrow@ICDCS 2018、RP-CH@JOS。


## Data sets & Experimental Design

<!-- 撰写实验环境的设置，实验的对象，实验的比较方面，以及实验的结果（不要列举数据，要概括谈） -->
测试平台：Amazon AWS 66个不同的云主机类型；测试应用：Spark、Hadoop 应用；实验对象：Exhaustive search、Coordinate descent、Random search、Ernest@NSDI 2016；评价指标：使用推荐云配置运行大数据应用的时间和成本。实验结果证明CherryPick相比于实验对象，有45-90%的概率找到更优云配置。


## Conclusion And Future Work

<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->
该论文面向大数据应用基于概率理论搜索最优云配置，想法令人眼前一亮，也是后续大量相关工作借鉴、比较的对象。
