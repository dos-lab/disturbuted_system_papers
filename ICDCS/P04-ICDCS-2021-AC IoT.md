# Title

Towards Efficient Inference: Adaptively Cooperate in Heterogeneous IoT Edge Cluster

## Website
https://doi.org/10.1109/ICDCS51616.2021.00011

## Citing

@INPROCEEDINGS{9546497,
author={Yang, Xiang and Qi, Qi and Wang, Jingyu and Guo, Song and Liao, Jianxin},
booktitle={2021 IEEE 41st International Conference on Distributed Computing Systems (ICDCS)},
title={Towards Efficient Inference: Adaptively Cooperate in Heterogeneous IoT Edge Cluster},
year={2021},
pages={12-23}

## Brief Introduction

该论文提出了一种应用于物联网下边缘计算场景的、基于流水线的 CNN 推理模型 PICO（Pipelined Cooperation）。通过将设备集群分成多个子集（stage）的形式，根据负载动态地选择推理策略，旨在满足延迟要求的前提下，最小化流水线周期以提高吞吐率。经试验，该模型在不同的工作负载下，推理时延平均降低了1.7-6.5倍，吞吐率提高了1.8-6.2倍。

## Key Methodology

- 基于流水线的 CNN

  对于 CNN 输出的特征映射的计算，将其分为多个部分，并分别使用不同的边缘设备进行计算。一个完整的特征映射的输出作为流水线的一个完整的任务，而特征映射的各部分计算则为流水线上的一个步骤。该论文使用融合特征级别（fused-layer）的并行，与单层级别的并行相比，可以降低设备间的通信代价，提高分割性能。

- 问题定义

  对于给定的 CNN 模型 *M* 与异构集群 *D*，找到一种 stage 分隔方式 *S*（每个 stage 为集群的一个子群，计算 CNN 模型内连续的若干层），使流水线的时延小于最大时延限制，并且流水线周期最短。

- 代价模型

  - 计算代价

    对于卷积层 *l<sub>i</sub>* ，设其特征映射 *F<sub>i</sub><sup>k</sup>* 通道数为 *c<sub>i</sub>* ，宽、高为 *w<sub>i</sub>*、*h<sub>i</sub>* ，卷积核大小为 *k<sub>i</sub>* * *k<sub>i</sub>* ，则其FLOPs（每秒浮点运算次数）为
    $$
    f(l_i; F^k_i)=k_i^2c_{i-1}w_ih_ic_i
    $$
    其中，每层的宽与高都可以由下一层的宽、高、步幅与卷积核大小递归计算得到。而对于异构设备集群 *D* 中的设备 *d<sub>k</sub>* ，其计算模型 *M* 的第 *i* 到第 *j* 层 *M<sub>i→j</sub>* 所对应的特征映射 *F<sub>k</sub><sup>j</sup>* ，FLOPs 即为各层的 FLOPs 之和。在此基础上，设备 *d<sub>k</sub>* 的推理时间即为 FLOPs 与设备计算性能之比。而对于参与 *M<sub>i→j</sub>* 计算的所有设备 *D<sub>i→j</sub>* ，其流水线周期即为所有设备推理时间的最大值。

  - 通信代价
  
    对于输入的特征映射 *F<sub>i</sub><sup>k</sup>* 与输出的特征映射 *F<sub>i</sub><sup>k</sup>* ，设网络带宽为 *b*，且假设在同一无线局域网环境内各设备带宽一致，则通信代价为输入与输出特征映射大小之和与带宽之比。则对于 stage 内的所有设备，通信代价为所有设备各层的通信代价之和。
  
  - 总代价
  
    对于一个 stage *S<sub>i→j</sub>* ，其总代价为计算代价与通信代价之和。则模型的优化目标为使流水线时延（各 stage 的总代价之和）小于最大时延限制，同时最小化 stage 集合的最大总代价。
  
- 启发函数
  
  由于模型要解决的是一个 NP 问题，因此先找出该模型在同构集群场景下的解，然后使用贪婪算法求异构集群场景下的解。
  
  将异构模型根据平均性能等效为同等性能的同构集群，则 stage 的分割问题可用动态规划解决。在使用动态规划确定 stage 的划分，进而确定 CNN 模型各连续层的划分后，保持对模型划分不变，使用贪心算法，使异构集群的各 stage 的 FLOPs 与同构集群对应 stage 的 FLOPs 尽可能接近。
  
- 一般化

  按照连续层对 CNN 模型划分的方法适用于链式结构的 CNN 模型，而对于如 ResNet34 等图式结构的 CNN 模型，则以 block 为单位划分为链式结构，然后将 block 视为层进行计算。

- 基于工作负载的自适应并行模式切换

  在实际中，系统的工作负载往往随时间发生较大变化。当负载较少时，可能只有几个甚至一个 stage 保持工作，造成了计算资源的浪费。在这种情况下，将切换至单 stage 模式，即将整个异构集群视为一个 stage 。这种情况可视为排队论中的 *M/D/1* 模型，因此可以直接代入公式计算两种模式的平均推理时延，并根据结果决定是否切换并行的推理模式。对于当前时刻的工作负载，使用移动平均法，根据上一时刻测量所得的工作负载进行估计。

## Data sets & Experimental Design

实验环境： 8 ARM based Raspberry-Pi 4Bs、WIFI with 50Mbps bandwidth

实验模型：VGG-16, YOLO, ResNet34, InceptionV3

- 与其他模式比较

  实验对象：Layer-wise(LW), Early-fused-layer(EFL), Optimal Fused-layer(OFL), Pipelined Cooperation(PICO)

  比较方面：最大吞吐率、平均推理时延、CPU 利用率、

  实验结果：

  在不同的模型、设备数与 CPU 频率下，PICO的推理周期均为最短，且吞吐率最高，其次为 OFL。

  在不同负载下，PICO 与可切换推理模式的 PICO（文中称为APICO）平均推理时间几乎没有变化，且在低负载时，APICO 推理时间低于 PICO。

  对于如 ResNet34、IncetionV3 等图式结构的 CNN 模型，PICO 有较高的加速比。

  PICO 在大部分 CPU 频率下的 CPU 利用率为最高，且平均利用率最高


## Conclusion And Future Work

本文针对物联网场景下 CNN 的边缘计算，提出了流水线模型 PICO，使用动态规划+贪婪算法逼近异构集群下子集群 stage 划分的最优解，并通过切换推理模式的方式应对动态负载。
