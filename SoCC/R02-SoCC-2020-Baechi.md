# Titile

@inproceedings{jeon2020baechi,
  title={Baechi: fast device placement of machine learning graphs},
  author={Jeon, Beomyeol and Cai, Linda and Srivastava, Pallavi and Jiang, Jintao and Ke, Xiaolan and Meng, Yitao and Xie, Cong and Gupta, Indranil},
  booktitle={Proceedings of the 11th ACM Symposium on Cloud Computing},
  pages={416--430},
  year={2020}
}

## Citing

Jeon, Beomyeol, et al. "Baechi: fast device placement of machine learning graphs." Proceedings of the 11th ACM Symposium on Cloud Computing. 2020.

## Brief introduction

针对深度学习模型且分时的模型并行问题进行改造优化，重点克服强化学习方法训练时间长的问题。

## Key Methology

采用两种典型策略：内存约束的低通讯切分；内存约束的早任务切分。比较直观的方法

## Data sets

Inception-V3 and Google
Neural Machine Translation System (GNMT).

## Experimental Design

1. baechi的放置时间；
2. 放置结果监测出来的机器学习模型迭代运行时间；
3. 当内存不足时，迭代运行时间收到的影响
4. 与单个GPU和专家放置结果的对比；
5. bachi自己优化策略产生的影响。

## Conclusion And Future Work

需要考虑如何尽快复现本文的工作，控制算子放置和模型并行。

1. 考虑异构资源；
2. 考虑多种机器学习模型的同时放置；
3. 考虑大内存对GPU受限内存造成的影响；
4. 考虑模型结构不断变化对放置造成的影响
5. 如何与上层的虚拟化技术结合
