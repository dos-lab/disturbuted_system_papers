# Title

<!-- 此部分是论文标题-->
Lorien: Efficient Deep Learning Workloads Delivery

## Website
<!-- 网址，有DOI的建议用DOI地址-->
https://dl.acm.org/doi/pdf/10.1145/3472883.3486973

## Citing

<!-- 引用格式，建议使用latex格式-->
@inproceedings{yu2021lorien,
  title={Lorien: Efficient Deep Learning Workloads Delivery},
  author={Yu, Cody Hao and Shi, Xingjian and Shen, Haichen and Chen, Zhi and Li, Mu and Wang, Yida},
  booktitle={Proceedings of the ACM Symposium on Cloud Computing},
  pages={18--32},
  year={2021}
}

## Brief Introduction

<!-- 通过三五句话描述这篇文章，包括 1. 论文的应用场景；2. 论文克服已有方法的局限性；3. 论文主要的技术手段； 4. 论文的预期结果 -->
深度学习模型的inference性能调优，如今大多数人都采用自动调度调优框架，这种方法更具有可扩展性。但通常它们的调优很慢。本文的Lorien基础架构，设计为在自动调优框架与计算资源之间的抽象层，提前地将大量的模型使用多种不同的调优框架，在各种类型计算资源上进行调优，将调优结果储存。当用户提交新的调优任务时，从数据库找出那些能够重复利用的调优结果，从而节省时间。若某调优任务在数据库中无法找到，则利用AutoML方法，依据之前的调优日志作为训练数据，训练得到一个调优结果的性能评估模型。通过随机采样schedule并使用该模型进行排序，尽可能快的为该调优任务生成一个近似最优的调优结果。

## Key Methodology

<!-- 分点写，论述论文中主要技术手段的实施过程 -->
定义调优的对象为workload，它通常为一个子图或算子。如：一个算子的workload调优（如AutoTVM），一个子图的workload调优（如Ansor）。

- 动机与挑战

    调优框架的调优速度通常要花数个小时，甚至一天，并且针对不同硬件的调优结果并不能够互相借用，不同平台的调优结果差距很大。为了让用户能够尽快得到调优结果，需要将大量的模型workloads进行调优，并且存入数据库。但这有以下挑战：

    - 调优进程耗时较长，一般的调优框架没有做错误容忍，以及针对多机器同时调优，需要在高层进行进一步管理。
    - 调优结果管理复杂：每个调优框架都采取基于文件的方式储存调优结果，但是每个调优框架的结果格式不尽相同，需要高层管理。
    - 储存全部的workload调优结果是不现实的，一定会出现用户需要调优的workload不存在的情况。

- 架构

    - tuning task generator:
        
        生成一系列调优任务，避免调优重复的任务。

    - distributed tuner：
        
        将调优任务调度到不同的云实例或边缘设备上。
        保障调优环境一致性以及调优稳定性。

    - model builder：
        
        查询最优的调优结果，构建二进制模型。

    - performance cost model trainer：
        
        使用调优的日志数据作为训练数据，训练一个性能代价模型。当未来的一个未知的workload到来时，可以使用它快速得到一个性能较好的调优结果。

- tuning task generator

    调优任务生成器，接受一个模型文件，对它进行解析（如TVM解析到Relay IR），抽取出一系列调优任务。

- distributed tuner

    分布式调优器：从tuning task generator得到一批调优任务，调优器将它们分布到实际机器上进行调优。采取主从架构，Master将任务调度到workers上，维护调优状态。master节点首先接受调优任务信息，随后启动任务管理器，跟踪记录进度。master节点采取无状态设计，能够随时进行灾备。

    在云上调优：现在的云服务通常提供批处理服务，简单地将调优任务提交给该服务，定期检查日志跟进状态即可。

    在边缘设备调优：需要构建一套device farm。采取了边缘设备主动向master拉取任务的架构，方便边缘设备随时退出集群。

- data model：
  
    使用DynamoDB noSQL储存调优结果，将调优结果使用二进制储存，保证每种框架的结果都能存储。一个workload以及它针对调优的目标平台合起来作为一个唯一的key。同时储存调优框架以及目标平台（如CUDA）等的版本号。

- model builder：

    从头将一个模型文件拆分成tuning task，从数据库中寻找对应的调优结果。如果都找到了，就将调优结果合并进行编译，生成二进制的可部署文件。若有找不到的tuning task，就用Zero-Shot tuning方法

- zero-shot tuning：

    workload不可能在数据库中全部记录：如某个Conv2D算子，有13个参数，有一个不一样，就需要重新调优。现有的自动调优框架都使用了性能估计模型，预测给定schedule的性能，避免在机器上的测量。但是自动调优框架里面的模型都是针对单一目标平台的，便携性差。并且这些模型都是需要在机器上进行不断地调优，在设备上测量时间还是他们的瓶颈。

    Lorien使用AutoML，使用之前在其他模型的调优历史日志，进行训练。使用每个schedule结果的高层特征作为训练数据。得到一个精确并且便携的从高层特征进行性能评估的模型是很难的，这是因为模型对底层硬件特征并不了解，只能通过数据进行了解。但实验表明，训练数据足够多时，准确度还可以。

    工作流：当发现一个missing的workload时，Lorien从抽样数千个scheduling，随后用AutoML模型进行排序。最终在设备上评估排名靠前的几个schedule。

## Data sets & Experimental Design

<!-- 撰写实验环境的设置，实验的对象，实验的比较方面，以及实验的结果（不要列举数据，要概括谈） -->
只使用了AutoTVM和Ansor作为调优框架。

- 对调优结果进行分析
- AutoML模型的性能评估分析


## Conclusion And Future Work

<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->
Lorien除了让用户快速生成调优结果外，它内部的大量调优任务产生的大量日志信息可以进一步进行分析，并帮助生成了一个高层的性能评估模型。

Lorien数据库为算子级别的性能做了benchmarking。

最近流行的Neural architecture search（NAS)是为了寻找在某个下游硬件上具有更高性能和低延迟/内存占用而来的。Lorien能够帮助这些硬件资源感知的NAS算法构建细粒度的以及精确的性能模型。