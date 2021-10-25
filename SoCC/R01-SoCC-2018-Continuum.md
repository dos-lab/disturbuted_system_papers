# Titile

Continuum: A Platform for Cost-Aware, Low-Latency Continual Learning

## Citing

Tian H, Yu M, Wang W. Continuum: A Platform for Cost-Aware, Low-Latency Continual Learning[C]//Proceedings of the ACM Symposium on Cloud Computing. ACM, 2018: 26-40.

## Brief introduction
这篇文章从TensorFlow、XGBoost等不同特征的ML框架出发，针对这些框架在线上学习（持续学习）场景下ML模型更新困难、资源开销大的问题，设置了统一的适配层Continuum，以及通过设置不同策略（成本感知、尽力交付和用户自定义）来控制模型更新的成本和效率，是首次提出集成ML框架的系统，能够达到降低数据协作延迟、低成本训练费用和提升模型质量的问题，同时具有一定的伸缩性和容错性

## Key Methology

比较训练模型的质量，数据延迟，训练成本，训练速度
本论文的关键的贡献在于两种策略对成本和效率调度约束：
先验条件：
训练时间和训练数据集的数据量呈线性相关关系，公式3
（1）训练效率优先（try to abort）：
如果当前训练正在进行，然后新数据准备到来，此时有两种考虑：如果当前训练刚开始，且新数据比较大，将两次更新的数据合并效果更好，就中断当前训练；否则训练继续。公式六
（2）成本优先（try to update earlier）：
在任意时刻，需要决定是否执行模型更新，较少的模型更新和较短的训练时间可以控制成本，在数据完全可用前可以选择一个时机进行提前模型更新（如果提前更新比全量更新更快）；公式八

## Data sets
均是开源数据集Twitter，Criteo，MovieLens 聚焦三类主要算法：LDA，PageRank等

## Experimental Design


## Conclusion And Future Work

其核心是通过控制模型更新的时机来提升在线学习场景各类ML framework的准确度、成本和效率等方面，本质也是通过系统调度来控制，如果要改进，需要从几个方面入手：
除了更新时机外，能不能控制更多的内容，数据量，框架选型、迁移、并行度等；
能否有根据数据特征自主选择优化策略；
能否和SLAQ这些内容（框架内优化）进行结合，形成体系。
