# Title

Auto-tuning Parameter Choices in HPC Applications using Bayesian Optimization

## Website

https://ieeexplore.ieee.org/document/9139814


## Citing

 @INPROCEEDINGS{9139814,  author={Menon, Harshitha and Bhatele, Abhinav and Gamblin, Todd},  booktitle={2020 IEEE International Parallel and Distributed Processing Symposium (IPDPS)},   title={Auto-tuning Parameter Choices in HPC Applications using Bayesian Optimization},   year={2020},  volume={},  number={},  pages={831-840},  doi={10.1109/IPDPS47924.2020.00090}}

## Brief Introduction

该论文在高性能计算应用（HPC）场景下，使用基于 BO 的 HiPerBOt 进行资源参数（包括编译、运行时参数与应用级别的配置选项）预测，支持迁移学习，以使用成本较低的源数据预测目标数据，适用于资源受限的情况。

## Key Methodology

- BO

  对于求目标函数 $f(x)$ 的最小值，选取一个计算成本更低的替代函数 $L(x)$。后验分布 $p_{y|x}(y|x)$ ，即在 $x$ 值处目标函数值大于 $y$ 的概率。实际计算中，基于先验分布 $p_y(y)$ 与后验分布，求得$y$ 与 $x$ 的似然 $p_{x|y}(x|y)$ 。预先设定一个目标值 $y^{(τ)}$ ，比较 $y$ 与 $y^{(τ)}$ 的大小关系，可以将似然分为 $p_g(x)$ （小于阈值的情况）与 $p_b(x)$ （大于等于阈值的情况）。

  对于 $y^{(τ)}$ 的选择，考虑到稳定性，不采用观测历史的最佳值，而是取百分比前 $α$ 的值。最终得到替代函数：

  $L(x,y^{(τ)})={1 \over α+{p_b(x) \over p_g(x)}(1-α)}$

- HiPerBOt

  使用向量 $\vec{x}=[x_1,...,x_n]$ 表示由 $n$ 个配置组成的一个配置组合。通过20次随机采样建立初始观测集 $H_0$，然后进行迭代采样，在每一次迭代中，根据替代函数，根据$p_g(\vec{x})$寻找对函数值优化最大的 $\vec{x}$，并将结果并入观测集中，建立并更新$p_g(\vec{x})$ 和 $p_b(\vec{x})$ 两个概率分布函数。对于$p_g(\vec{x})$ 和 $p_b(\vec{x})$，按照配置进行分解：

  $p_g(\vec{x})=p_{g,x_1}(x_1)...p_{g,x_n}(x_n)$

  $p_b(\vec{x})=p_{b,x_1}(x_1)...p_{b,x_n}(x_n)$

  其中，对于离散参数，遍历观测集中对应参数以估计其 $p_{g,x_i}(x_i)$ 与 $p_{b,x_i}(x_i)$；对于连续参数，使用核密度估计（KDE）进行预测。

  在进行规定次数迭代或者迭代间优化效果小于阈值时，结束迭代。

- 迁移学习

  使用已有数据 $D^{Src}$ 作为先验分布以预测目标数据 $D^{Trgt}$，并在之后的迭代中将来自已有数据的概率分布进行加权，加到目标数据的概率分布中。目的是使用已有的、计算成本较小的配置数据去预测资源需求较高的配置数据。


## Data sets & Experimental Design

- 实验对象

  使用的高性能计算应用：Kripke、HYPRE、LULESH、OpenAtom

  比较对象：GEIST、随机选择、穷举

- 指标

  最优配置性能的执行时间、召回率（选出的 good 配置占总 good 配置之比）

- 实验结果

  - Kripke

    HiPerBOt 的执行时间比 GEIST 低 26%，且在96次采样后达到最优配置，召回率是 GEIST的2倍。

  - HYPRE

    搜索了配置空间的5%后达到最优配置，但所有方法在 HYPRE 的召回率都比较低（最高为50%）

  - LULESH

    达到最优配置的次数与性能差距不大，但召回率是 GEIST 的2倍。

  - OpenAtom

    搜索了配置空间的3%后达到最优配置，召回率比 GEIST 高出30%。

  - 初始采样数与 good 的阈值

    采样数为20时性能最佳，阈值为20%时性能最佳（均以执行时间衡量）。

- 参数配置的具体选择

  $p_{g,x_i}(x_i)$ 与 $p_{b,x_i}(x_i)$ 之间的差距越大，则代表该配置对总体应用性能的影响越大。使用 JS 散度来衡量两个分布之间的差异，并由该差异得到各个参数配置对应用性能影响的权重。 


## Conclusion And Future Work

该论文基于 BO，将配置组合分解为单个配置，并对其依次应用 BO，结合迁移学习以达到对高性能计算应用配置预测的目的。

