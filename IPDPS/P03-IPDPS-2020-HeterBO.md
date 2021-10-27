# Title

<!-- 此部分是论文标题及其引用格式，建议使用latex格式 -->
Not All Explorations Are Equal: Harnessing Heterogeneous Profiling Cost for Efficient MLaaS Training

## Website

https://doi.org/10.1109/IPDPS47924.2020.00051

## Citing

Yi J, Zhang C, Wang W, et al. Not All Explorations Are Equal: Harnessing Heterogeneous Profiling Cost for Efficient MLaaS Training[C]//2020 IEEE International Parallel and Distributed Processing Symposium (IPDPS). IEEE, 2020: 419-428.

## Brief introduction

<!-- 通过三五句话描述这篇文章，包括 1. 论文的应用场景；2. 论文克服已有方法的局限性；3. 论文主要的技术手段； 4. 论文的预期结果 -->
该论文针对深度学习应用在公有云场景下训练模型时需要推荐优化的云资源（云主机类型和云主机个数）以满足租用资源的时间/成本约束的问题，论述了相关工作在解决该问题时或需要收集大量数据而导致开销过大（性能建模）或未统筹考虑深度学习应用特征而导致准确性不足（传统的贝叶斯优化）。为此，本论文提出了面向深度学习应用的贝叶斯优化资源供给方法--HeterBO，该方法结合了深度学习应用训练阶段的资源使用特性，能够减小深度学习应用训练的时间和成本。

## Key Methology

<!-- 分点写，论述论文中主要技术手段的实施过程 -->
该论文的核心发现是深度学习应用在训练阶段的资源使用存在典型特征（Fig. 3），并不是“资源越多性能越好”、“资源越贵性能越好”，而是存在明显的拐点。传统的贝叶斯优化算法在选择云资源时未统筹考虑上述特征，因而存在云资源推荐准确性低、成本高的问题。为此，该论文提出了改进的贝叶斯优化方法HeterBO，以权重的方式对传统贝叶斯优化算法的选择函数（EI）进行改进（TEI，公式 5），能够有效减小深度学习应用训练的时间和成本。


## Data sets & Experimental Design

<!-- 撰写实验环境的设置，实验的对象，实验的比较方面，以及实验的结果（不要列举数据，要概括谈） -->
测试平台：Amazon AWS 50个不同的云主机类型、100台云主机；测试应用：深度学习应用AlexNet, ResNet, Inception-v3, Char, BERT通过TensorFlow和MXNet实现；实验对象：Paleo@ICLR 2017和CherryPick@NSDI 2017；评价指标：使用推荐云配置进行深度学习应用训练的时间和成本。实验结果证明HeterBO相比于Paleo和CherryPick分别优化了3.1倍和2.34倍。


## Conclusion And Future Work

<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->
该论文的首次将深度学习应用训练阶段的资源使用特征与贝叶斯优化方法结合在一起，具有借鉴意义。然而该方法只适用于深度学习应用数据切分场景（是指将数据均等切分，在集群中并行执行），未来考虑面向深度学习应用模型切分场景（是指模型切分成不同的子模型/算子，在集群中并行执行）优化云资源供给。
