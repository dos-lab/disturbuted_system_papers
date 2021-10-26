# Titile

@inproceedings{jiang2020unified,
  title={A Unified Architecture for Accelerating Distributed $\{$DNN$\}$ Training in Heterogeneous GPU/CPU Clusters},
  author={Jiang, Yimin and Zhu, Yibo and Lan, Chang and Yi, Bairen and Cui, Yong and Guo, Chuanxiong},
  booktitle={14th $\{$USENIX$\}$ Symposium on Operating Systems Design and Implementation ($\{$OSDI$\}$ 20)},
  pages={463--479},
  year={2020}
}

## Citing

Jiang, Yimin, et al. "A Unified Architecture for Accelerating Distributed {DNN} Training in Heterogeneous GPU/CPU Clusters." 14th {USENIX} Symposium on Operating Systems Design and Implementation ({OSDI} 20). 2020.

## Brief introduction

深度学习分布式并行训练时，All-reduce和PS模式都不能充分利用CPU资源。为了充分利用异构资源BytePS对参数同步服务进行分解，并优化通信服务和累加服务。

## Key Methology

BytePS能够实现额外CPU资源之间的互相通信，以充分利用网络资源；且通过分解运算到GPU、参数累加到CPU的方式，结合PCI、RDMA、NIC等网络拓扑实现接近理论最优的网络资源利用，实现高效的参数同步。

## Data sets

1. PCI、NVLINK不同连接拓扑下的性能
2.  ResNet-50， GPT-2，GG-16，UGATIT，Transformer，BERT-Large

## Experimental Design



## Conclusion And Future Work

感知数据量和通信方式，白盒方法。
