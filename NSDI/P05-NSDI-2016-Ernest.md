# Title

<!-- 此部分是论文标题及其引用格式，建议使用latex格式 -->
Ernest: Efficient Performance Prediction for Large-Scale Advanced Analytics

## Website

https://www.usenix.org/conference/nsdi16/technical-sessions/presentation/venkataraman

## Citing

Venkataraman S, Yang Z, Franklin M, et al. Ernest: Efficient performance prediction for large-scale advanced analytics[C]//13th {USENIX} Symposium on Networked Systems Design and Implementation ({NSDI} 16). 2016: 363-378.

## Brief Introduction

<!-- 通过三五句话描述这篇文章，包括 1. 论文的应用场景；2. 论文克服已有方法的局限性；3. 论文主要的技术手段； 4. 论文的预期结果 -->
该论文研究大规模数据分析应用的性能预测问题，为每一个应用自动选择合适的资源配置。首先，论文发现了大规模数据分析应用由于结构相似和可预测性。然后，根据统计数据将应用的结构划分成多对一、树状、多对多等3种交互模式的集合，并据此建立线性回归方程。最后，利用离线分析过程的小规模数据采样对模型进行训练，从而构建配置与性能之间的精确预测模型。

## Key Methodology

<!-- 分点写，论述论文中主要技术手段的实施过程 -->
该论文的核心贡献是统计了大规模数据分析应用的三种数据交互模式，其建立的性能预测模型能够精确预测应用在不同资源配置下的性能。


## Data Sets & Experimental Design

<!-- 撰写实验环境的设置，实验的对象，实验的比较方面，以及实验的结果（不要列举数据，要概括谈） -->
测试平台：Amazon AWS；测试应用：大规模数据分析应用；比较对象：cost-based方法；评价指标：预测性能与真实性能的误差。实验结果表明Enrest能够减小30-50%的预测误差。


## Conclusion and Future Work

<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->
该论文对于三种数据交互模式的分析具有借鉴意义，未来可以在不同场景中借鉴这种思想，例如将深度学习应用划分成几种主要的数据交互模式，用于精确预测性能。
