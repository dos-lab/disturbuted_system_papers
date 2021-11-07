# Title

Generating Complex, Realistic Cloud Workloads Using Recurrent Neural Networks

## Website
https://doi.org/10.1145/3477132.3483590

## Citing

@inproceedings{10.1145/3477132.3483590,
author = {Bergsma, Shane and Zeyl, Timothy and Senderovich, Arik and Beck, J. Christopher},
title = {Generating Complex, Realistic Cloud Workloads Using Recurrent Neural Networks},
year = {2021},
booktitle = {Proceedings of the ACM SIGOPS 28th Symposium on Operating Systems Principles},
pages = {376–391},
numpages = {16},
keywords = {deep learning, cloud workload modeling, survival analysis, trace generation, recurrent neural networks},
series = {SOSP '21}
}

## Brief Introduction

该论文针对大规模云计算场景提出了相应的基于 RNN 的大规模云负载模型，首次揭示了在长程任务中，负载的到达率（泊松回归）、资源请求（LSTM）与生命周期（LSTM）间的相互关系，进而进行调度优化。论文对这三种特征依次进行了共三阶段生成模型的建模，在此基础上生成 trace，并验证了这种方式可以在两种云供应商场景（微软的 Azure 和华为云）下精确生成虚拟机负载。

## Key Methodology

1、到达模型，使用泊松回归模型，描述一段时间内batch（在同一时间段内来自同一用户的任务集合）数量的分布情况，并使用负对数似然作为损失函数。模型特征为使用独热编码与 survival encode 的、以小时与天为单位的粗粒度时序信息。

2、资源模型，基于LSTM，预测一段时间内所有任务的资源请求，以 flavor（对特定系列资源的请求） 序列的形式呈现。该模型输出 K 个可能的 flavor 分布与结束符 EOB，并通过 softmax 函数输出为相对应的多项分布。模型特征为经过独热编码的上一时间步的 flavor （没有则为EOB），以及到达模型中使用的时序信息特征。

3、生命周期模型，基于LSTM，返回资源序列中每个任务可能的生命周期的分布。该模型输出任务的若干个可能生命周期的 bin，使用 hazard 概率求得生命周期结束于某个 bin 的可能性。模型特征有五个，前四个分别是1中的时序特征、2中的当前任务的资源请求、当前 batch 的任务数、之前任务的生命周期（survival-encoding），最后一个特征用来描述任务的生命周期是否右删失（即是否在观测窗的结尾前终止）。

4、将以上三者综合为带有资源请求的任务开始 / 结束时间以生成 trace，其中，使用连续密度插值法（continuous-density interpolation）将离散的生命周期 bin 转化为真实的持续时间。


## Data sets & Experimental Design

数据：Microsoft Azure data（AzurePublicDatasetV1） & Huawei Cloud data

1、到达率：对于每个测试时间段，根据泊松分布得到预测可能性大于90%的区间，计算实际数据落在该区间内的概率。对于本文提出的 batch 到达模型，使用了不包含 / 包含 DOH（Day-of-History） 时序特征的任务到达模型进行对比，证明使用带有 DOH 时序特征的 batch 到达模型的性能更好。

2、资源请求（flavor）：比较与 baseline（Uniform、Multinomial、RepeatFlav）的预测效果（即从所有 flavor 的可能选择中进行预测），使用可能性的负对数似然与下一步最佳分类错误率（1-Best-Err）为指标，证明 LSTM 模型从这两个指标上看优于三种 baseline。

3、生命周期：比较与 baseline（CoinFlip、Overall KM、Per-flavor KM、RepeatLifetime）对单独的生命周期 bin 的预测准确率，使用二值交叉熵与 1-Best-Err 为指标。然后，基于连续值的 Survival-MSE 评估，将模型离散的 hazard 输出转化为连续值的生存函数，与每个任务的实际的生存函数进行比较。

4、在实际的应用场景中，选取了产能规划、负载调度两个场景，并以 NAIVE、SIMPLEBATCH 作为 baseline。对于产能规划场景，预测了测试窗口内每一时刻活跃的总 CPU 数量，并与 baseline 进行对比。对于负载调度场景，使用 reuse distance（复用距离）、packing fragmentation（包的碎片化程度）。


## Conclusion And Future Work

本文首次提出了一种揭示长程任务中到达率、flavor生命周期三者相互关系的大规模云负载模型，并且能高质量地生成工作负载，进而验证了其在实际的中程产能规划、负载调度优化的应用。该文章面向的是同构集群支持的云计算场景，并没有考虑在异构资源中的应用情况。
