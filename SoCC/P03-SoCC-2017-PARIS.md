# Title

<!-- 此部分是论文标题及其引用格式，建议使用latex格式 -->
Selecting the Best VM across Multiple Public Clouds: A Data-Driven Performance Modeling Approach

## Website

https://doi.org/10.1145/3127479.3131614

## Citing

Yadwadkar N J, Hariharan B, Gonzalez J E, et al. Selecting the best vm across multiple public clouds: A data-driven performance modeling approach[C]//Proceedings of the 2017 Symposium on Cloud Computing. 2017: 452-465.

## Brief Introduction

<!-- 通过三五句话描述这篇文章，包括 1. 论文的应用场景；2. 论文克服已有方法的局限性；3. 论文主要的技术手段； 4. 论文的预期结果 -->
该论文主要面向跨公有云平台的大数据应用云配置（云主机类型+云主机个数）优化问题，首先通过实验发现跨公有云平台，大数据应用运行在相近的云配置的性能相差很大。然后，分析了这种现象将导致已有性能建模方法（Ernest@NSDI 2016）和黑盒搜索方法（CherryPick@NSDI 2017）失效。最后，提出了数据驱动的两阶段云配置优化方法PARIS，离线阶段通过收集大量的离线测试数据训练随机森林模型，在线阶段通过训练好的模型预测目标大数据应用优化的云配置。

## Key Methodology

<!-- 分点写，论述论文中主要技术手段的实施过程 -->
该论文主要面向SQL-like类型的大数据应用，通过离线收集MongoDB等应用的大量数据建立决策树，并充分利用随机森林的集成学习（Ensemble learning）思想构建性能预测模型。然而，PARIS的离线训练阶段仅考虑收集底层资源使用数据，当用于预测非SQL-like类型应用时，由于底层资源使用数据的相似性不高而存在模型失效的风险。


## Data Sets & Experimental Design

<!-- 撰写实验环境的设置，实验的对象，实验的比较方面，以及实验的结果（不要列举数据，要概括谈） -->
测试平台：Amazon AWS、Microsoft Azure；测试应用：SQL-like类型的大数据应用；实验对象：3个Baseline，详见论文；评价指标：使用优化的云配置运行大数据应用的时间和成本。实验结果证明PARIS能够有效减少45%的成本。


## Conclusion and Future Work

<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->
该论文贡献之一是发现了大数据应用在不同公有云的相似云配置中运行性能差异性，基于此采用经典的两阶段（离线训练+在线预测）优化方法比较直观。未来可以借鉴论文的套路，即发现一种场景是当前相关工作不适用的，进而提出新的模型、机制、策略。
