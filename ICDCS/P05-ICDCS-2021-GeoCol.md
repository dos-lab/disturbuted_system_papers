# Title

GeoCol: A Geo-distributed Cloud Storage System with Low Cost and Latency using Reinforcement Learning

## Website
https://doi.org/10.1109/ICDCS51616.2021.00023

## Citing

@INPROCEEDINGS{9546510,
author={Wang, Haoyu and Shen, Haiying and Li, Zijian and Tian, Shuhao},
booktitle={2021 IEEE 41st International Conference on Distributed Computing Systems (ICDCS)}, 
title={GeoCol: A Geo-distributed Cloud Storage System with Low Cost and Latency using Reinforcement Learning}, 
year={2021},
pages={149-159}

## Brief Introduction

该论文提出了一种基于强化学习的低成本低时延的跨地域云存储系统 GeoCol：①使用 SARIMA 预测请求时延，以此对请求进行分割，分别发送到对应的数据中心，以达到数据对象并行传输的目的。②对于每个数据中心，使用强化学习模型预测其存储的数据对象及其存储类型。通过以上两点，减少因存储过多数据副本而造成的存储冗余以及请求发送的冗余。该系统最高能降低32%的成本开销以及51%的数据对象请求时延。

## Key Methodology

1. 基于强化学习模型的自适应请求分割

   - 请求时延预测

     由请求分割代理对每个数据中心的请求时延进行预测。请求分割代理从网络监测器（本文假设每个服务器都有一个专门负责收集如请求时延等请求信息的网络监测器）获取请求数据，并且基于历史数据，对数据中心的时延进行周期性的预测。其中，为了减少网络监测的开销，当数据中心五分钟内没有请求时，监测器获取一次数据中心的时延信息（“最后请求”，即监测器激活前的最后一个请求）

     该论文通过实验证明，由于 SARIMA 模型能更好地掌握时间序列数据的周期性特征，因此预测准确率比其他模型更好，因此该论文使用 SARIMA 模型对请求时延进行预测。针对每个数据中心建立一个预测模型会显著提高模型的构建、训练与维护代价，因此该论文对所有数据中心使用一个预测模型，并将所有数据中心的数据作为训练数据，而每个数据中心的位置作为模型的输入特征。为了避免预测时间过长，模型根据数据中心的情况，每次进行特定长度序列的预测。

     模型的输入为当日内被请求的数据对象的大小、目的数据中心的位置；模型的输出请求为设定的周期时间上、固定大小数据对象的请求到达每个数据中心的时延。
     
   - 基于强化学习的请求分割

     - 状态空间

       包括①每个数据中心的最后请求  ②每个数据中心的预测请求时延  ③当前请求的特征组成，包括每个数据中心的位置、最后请求的时延以及请求数据对象的大小

     - 动作空间

       一个请求被分割成子请求的数量 *n*，及这 *n* 个子请求的分别的目的地。

     - 奖励函数

       三个特征的加权和：实际请求时延对应的请求数据对象大小、是否完成时延目标、成本支出的倒数。其中成本由请求的数据对象的对应种类及所需数量的数据单元成本（存储成本）与数据传输成本两者组成。

2. 基于强化学习模型的数据存储与类型选择

   - 状态空间

     即数据中心中 web 应用的所有数据对象的请求信息，其中包括：数据对象是否存储在当前数据中心（一个标记，存在则为1，否则为0）、请求频率、数据对象大小、平均时延达成率、平均请求时延、数据对象存储类型

   - 动作空间

     数据对象是否存储在当前数据中心、数据对象存储类型。其中，如果标记为1但是当前数据中心没有存储数据对象的副本，则将会从最近的存储该数据对象副本的数据中心获得一个副本。

   - 奖励函数

     所有数据中心的平均时延达成率、平均请求时延、总成本的倒数、时延与成本的权衡系数的四者的乘积的和。


## Data sets & Experimental Design

实验环境： datacenters in US-east, US-west, AP (Asia Pacific)-southeast, AP-northeast, EU-west, and CA (Central Asia)-central in AWS S3

基线：CosTLO、TAILCUTTER、Prop、SPANStore、SEDPT

实验数据：Wikipedia trace

比较方面：请求时延、SLO 达成率、成本

实验结果：对于不同大小的数据对象，GeoCol 的平均请求时延最低、SLO 达成率最高。对于不同的使用时间，GeoCol 的成本最低，


## Conclusion And Future Work

该论文使用强化计算对请求时延及数据存储进行预测，以降低成本和时延，感觉这种思路可以用在对边缘计算的优化上。
