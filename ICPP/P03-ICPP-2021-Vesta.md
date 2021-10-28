# Title

<!-- 此部分是论文标题及其引用格式，建议使用latex格式 -->
Best VM Selection for Big Data Applications across Multiple Frameworks by Transfer Learning

## Website

https://doi.org/10.1145/3472456.3472488

## Citing

Yuewen Wu, Heng Wu, Yuanjia Xu, Yi Hu, Wenbo Zhang, Hua Zhong, and Tao Huang. 2021. Best VM Selection for Big Data Applications across Multiple Frameworks by Transfer Learning. In 50th International Conference on Parallel Processing (ICPP '21), August 9–12, 2021, Lemont, IL, USA. ACM, New York, NY, USA, 11 pages.

## Brief introduction

<!-- 通过三五句话描述这篇文章，包括 1. 论文的应用场景；2. 论文克服已有方法的局限性；3. 论文主要的技术手段； 4. 论文的预期结果 -->
当前用户由于业务需求通常选择多个大数据框架进行数据分析，且主流的大数据框架有数十种之多，云配置择优面临框架异构性的挑战。已有机器学习相关工作（PARIS@SoCC 2017）大多复用已学习应用构建的模型（应用来源于一个或几个大数据框架）进行云配置择优，然而此方法并不适用于大数据应用跨框架的场景，将已学习应用构建的模型直接复用于未学习大数据框架之上将导致严重的预测误差，而重新构建模型可能需要数百小时的数据收集、模型训练开销。另一方面，对于执行相同或相似任务的大数据应用，即使存在框架上的差异，应用之间也可能体现出相同或相似的资源关系（Figure 1），进而具有云配置择优知识互通与共享的可行性。为此，本论文通过大规模离线分析，深度挖掘跨框架大数据应用的云配置择优知识并构建知识共享模型，采用迁移学习方法实现知识在不同框架的迁移，兼顾模型预测的准确性和模型构建的开销。

## Key Methology

<!-- 分点写，论述论文中主要技术手段的实施过程 -->
该论文的核心贡献在于发现、表示、复用跨框架大数据应用的云配置择优知识，解决了已有工作难以兼顾模型预测准确性和模型构建开销的问题。


## Data sets & Experimental Design

<!-- 撰写实验环境的设置，实验的对象，实验的比较方面，以及实验的结果（不要列举数据，要概括谈） -->
测试平台：Amazon AWS 120个不同的云主机类型；测试应用：三个大数据框架（Spark、Hadoop、Hive）的30个应用；实验对象：PARIS@SoCC 2017、Ernest@NSDI 2016；评价指标：大数据应用在推荐云配置运行时的性能提升，以及获取推荐云配置产生的训练开销。实验结果表明Vesta相比于实验对象，Vesta能够提升51%的应用性能，并且减少85%的训练开销。


## Conclusion And Future Work

<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->
该论文基于迁移学习解决跨框架大数据应用的云配置择优问题，其大规模离线测试的方法值得借鉴，可用于分析其他主流应用的资源使用特征，例如PyTorch、TensorFlow等深度学习应用。
