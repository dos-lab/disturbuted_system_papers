# Title

<!-- 此部分是论文标题及其引用格式，建议使用latex格式 -->
CherryPick: Adaptively Unearthing the Best Cloud Configurations for Big Data Analytics

## Website

https://www.usenix.org/conference/nsdi17/technical-sessions/presentation/alipourfard

## Citing

Alipourfard O, Liu H H, Chen J, et al. Cherrypick: Adaptively unearthing the best cloud configurations for big data analytics[C]//14th {USENIX} Symposium on Networked Systems Design and Implementation ({NSDI} 17). 2017: 469-482.

## Brief Introduction

<!-- 通过三五句话描述这篇文章，包括 1. 论文的应用场景；2. 论文克服已有方法的局限性；3. 论文主要的技术手段； 4. 论文的预期结果 -->
该论文面向周期性大数据应用在公有云计算环境中运行时的云配置优化问题，开创性地采用贝叶斯优化（Bayesian Optimization）方法搜索最优云配置（云配置指云主机类型、云主机个数）。论文提出当前云配置优化系统大多基于性能建模（例如Ernest@NSDI 2016）方法，难以兼顾模型构建的准确性、开销与适用性。为此，该论文基于概率理论将云配置优化问题转化为黑盒搜索问题，通过周期性大数据应用自身迭代过程持续搜索优化的云配置，直到达到预期的优化效果（公式 1：保障应用执行时间的同时尽可能减小云配置开销）。

## Key Methodology

<!-- 分点写，论述论文中主要技术手段的实施过程 -->
该论文的贡献在于开创性地采用概率搜索方法来解决云配置优化问题，实现的贝叶斯优化方法原理直观有效（Figure 5），相比于当时主流的性能建模方法优势在于能够有效解决建模开销大、适用性差的问题。CherryPick概率搜索方法的局限性在于仅适用于周期性大数据应用。后续国内外相关研究方向有很多论文都借鉴、改进了CherryPick的实现，例如：HeterBO@IPDPS 2020、Arrow@ICDCS 2018、RP-CH@JOS。

- 问题定义

   设 $T(\mathbf{x})$ 为配置 $\mathbf{x}$ 下应用在其输入负载下的运行时间，而 $P(\mathbf{x})$ 为配置 $\mathbf{x}$ 下所有虚拟机每单位时间价格，则问题可以定义为使 $T(\mathbf{x})$ 在最大容许运行时间内，同时使 $T(\mathbf{x})$ 与 $P(\mathbf{x})$ 的乘积$C(\mathbf{x})$ （假定为高斯过程）最小。

- 贝叶斯优化过程

   在 $C(\mathbf{x})$ 未知且无法求导时，对其进行有限次数的采样，并计算采样点之间的函数值的置信区间，并使用采集函数（acquisition function）确定下一个采样点，即使采集函数值最大的 $\mathbf{x}$ 即为下一个采样点，将问题转化为对采集函数取最大值。

- 采集函数：Expected Improvement (EI)
  
  选取使当前时刻采集函数值提升程度最大的 $\mathbf{x}$ 。其效用函数可表示为 
  $$
  I(\mathbf{x})=max(y'-y,0)
  $$
  其中，当前 $\mathbf{x}$ 对应值 $y$ ，下一时刻最小值 $y'$ 发生在点 $\mathbf{x'}$ 。假设 $y$ ~ $N(μ,σ^2)$ ，再考虑到运行时间要求，最终有
  $$
  EI(\mathbf{x})=P[T(\mathbf{x}≤T_{max})]*E_{y~N(μ,σ^2)}[I(\mathbf{x})]
  $$
  
  根据该采集函数，当尝试了若干种不同的云配置后，函数值提升小于预设阈值，则终止过程。
  
- 应对不确定性
  
  在实际情况中，由于多种原因（如资源失效或过载、用户间相互干扰等），会导致实际的运行时间与估计值间有一定百分比的误差，表示为：
  $$
  \widetilde{T}(\mathbf{x})=T(\mathbf{x})(1+ε_c)  \\\widetilde{C}(\mathbf{x})=C(\mathbf{x})(1+ε_c)
  $$
  其中， $ε_c$ 为噪音，服从正态分布：$ε_c$ ~ $N(0,σ^2_{ε_c})$ 。由于相乘的两者都服从正态分布，因此结果并不服从正态分布，不适用于贝叶斯优化。在此，对 $T(\mathbf{x})$ 与 $C(\mathbf{x})$ 取对数。由于 $ε_c$ 在0~1之间，因此 $log(1+ε_c)$  可近似为 $ε_c$， 则问题可重新适用于贝叶斯优化。


## Data Sets & Experimental Design

<!-- 撰写实验环境的设置，实验的对象，实验的比较方面，以及实验的结果（不要列举数据，要概括谈） -->
测试平台：Amazon AWS 66个不同的云主机类型；测试应用：Spark、Hadoop 应用；实验对象：Exhaustive search、Coordinate descent、Random search、Ernest@NSDI 2016；评价指标：使用推荐云配置运行大数据应用的时间和成本。实验结果证明CherryPick相比于实验对象，有45-90%的概率找到更优云配置。

- 实验环境

  基准应用：TPC-DS, TPC-H, TeraSort, The SparkReg, SparkKm

- 实验对象

  CherryPick、穷举法、坐标下降法、有限随机搜索、Ernest
  
- 比较方面

  在当前配置下的运行成本、搜索成本、模型优化程度

- 实验结果

  - 与穷举法相比，CherryPick 的运行成本与搜索成本之和最低
  - 与坐标下降法相比，CherryPick 能更稳定地找到（接近）最优解
  - 与有限随机搜索相比，在搜索代价相似时，CherryPick 更稳定地找到更优配置
  - 与 Ernest 相比，在运行成本相当时，CherryPick 的搜索成本与时间更低。



## Conclusion and Future Work

<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->
该论文面向大数据应用基于概率理论搜索最优云配置，想法令人眼前一亮，也是后续大量相关工作借鉴、比较的对象。
