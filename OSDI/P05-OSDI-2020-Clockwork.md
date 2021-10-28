# Title

@article{gujarati2020serving,
  title={Serving DNNs like Clockwork: Performance Predictability from the Bottom Up},
  author={Gujarati, Arpan and Karimi, Reza and Alzayat, Safya and Kaufmann, Antoine and Vigfusson, Ymir and Mace, Jonathan},
  journal={arXiv preprint arXiv:2006.02464},
  year={2020}
}

## Citing

Gujarati, A., Karimi, R., Alzayat, S., Kaufmann, A., Vigfusson, Y., & Mace, J. (2020). Serving DNNs like Clockwork: Performance Predictability from the Bottom Up. arXiv preprint arXiv:2006.02464.

## Brief Introduction

针对深度学习推理时服务质量难以有效保障的问题，提出了分层评估机器学习软件栈以提升服务质量预测准确度的的方法，通过对硬件资源、操作系统和深度学习进行综合评估，提升预测准确度。

## Key Methodology

主控制器+分散工作节点：克服内存缓存、硬件交互关系、外部影响三方面的问题。

## Data Sets

DenseNet [36]
DLA [75]
GoogLeNet [67]
Inception [68]
Xception [68]
MobilePose [33]
ResNeSt [78]
ResNet [30]
ResNet-v2 [31]
ResNeXt [73]
SENet [35]
TSN [70]
Wide ResNet [76]
Winograd [45]

## Experimental Design

1. clockwork能否同时调度上千个服务
2. clockwork预测准确度；
3. clockwork对不同服务的性能隔离情况
4. clockwork能否应对真实负载；
5. clockwork的伸缩性。

## Conclusion and Future Work

性能建模的深度（机器学习软硬件技术栈）和广度（异构资源情况下任务执行状态与资源使用状态）能否成为工作改造的瓶颈。

1. 智能预测gpu资源，忽略数据预处理和数据后续处理的情况；
2. 在共享资源环境下预测会存在一定问题，所有资源都必须尽力独占运行需要预测的深度学习服务。
