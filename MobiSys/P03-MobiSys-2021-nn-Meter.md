# Title

nn-Meter: Towards Accurate Latency Prediction of Deep-Learning Model Inference on Diverse Edge Devices

## Website
https://doi.org/10.1145/3458864.3467882

## Citing

@inproceedings{10.1145/3458864.3467882,
author = {Zhang, Li Lyna and Han, Shihao and Wei, Jianyu and Zheng, Ningxin and Cao, Ting and Yang, Yuqing and Liu, Yunxin},
title = {Nn-Meter: Towards Accurate Latency Prediction of Deep-Learning Model Inference on Diverse Edge Devices},
year = {2021},
isbn = {9781450384438},
doi = {10.1145/3458864.3467882},
pages = {81–93},
numpages = {13},
series = {MobiSys '21}
}

## Brief Introduction

该论文提出了一种通过模型推理进行分割，以预测边缘设备推理时延的系统 nn-Meter，通过将模型推理划分为若干个核（kernel），并对其配置进行自适应采样以对时延进行准确预测。和算子级别的解决方案相比，可以将计算图运行时优化对时延造成的影响考虑在内。

## Key Methodology

- Kernel Detection

  - 算子融合

    对于单入单出的算子，若两者单独计算时间大于融合时间与融合算子时间（与阈值系数相乘，文中是0.5）之和，则进行融合

    对于单入多出的算子不进行融合（考虑到循环依赖等）

    对于多入单出的算子，不融合&两个输入分别与输出算子融合，共有三种情况，取其中运行时间与实际最为接近的组合。

  - Kernel 搜索

    基于 DFS，从计算图的根算子开始，按照算子融合的规则进行搜索。

- Latency Predictor

  - Adaptive data sampling

    对于时延占比较大的 Conv 算子，output channel 对时延的变化在 GPU 和 VPU 上呈阶梯型分布，因此用随机采样建模并不准确，所以使用 adaptive data sampling：对于预测误差大于阈值的位置，进行关于 output channel 的细粒度采样。

  - 时延预测
  
    - Kernel 级别
  
      使用随机森林回归，以应对非线性的时延模型。
  
    - 模型级别
    
      所有 kernel 时延之和。


## Data sets & Experimental Design

- 实验设置

  三种基准方法：FLOPs, FLOPs+MAC, BRP-NAS

- 比较方面

  RMSE, RMSPE

- 端到端

  - 实验对象

    nn-Meter, AlexNets, VGGs, MobileNetv1s, MobileNetv2s, NASBench201

  - 实验结果
  
    nn-Meter 是唯一能在各个设备上实现准确预测的方法，但是在 VPU 上的预测准确率明显低于 CPU 与 GPU。
  
- 微基准测试
  
  - 实验对象
  
    核级预测 & 算子级预测、自适应采样 & 随机采样
  
  - 实验结果
  
    在所有设备上，核级预测均优于算子级预测，且算子级预测准确率在不同设备上稳定率较低。
  
    在所有设备上，自适应采样的 RMSE 更低，且准确率更高。在模型级别上，随机采样无法进行准确预测。


## Conclusion And Future Work

该论文提出了一种基于核级别的模型推理时延预测系统，基于算子融合进行核检测。但是该系统只能应用于同构集群，且无法应用于时序模型。
