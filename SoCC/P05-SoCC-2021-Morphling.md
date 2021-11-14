# Title

<!-- 此部分是论文标题-->
Morphling: Fast, Near-Optimal Auto-Configuration for Cloud-Native Model Serving

## Website
<!-- 网址，有DOI的建议用DOI地址-->
https://cse.hkust.edu.hk/~lyangbk/papers/morphling-socc21.pdf

## Citing

<!-- 引用格式，建议使用latex格式-->
@inproceedings{wang2021morphling,
  title={Morphling: Fast, Near-Optimal Auto-Configuration for Cloud-Native Model Serving},
  author={Wang, Luping and Yang, Lingyun and Yu, Yinghao and Wang, Wei and Li, Bo and Sun, Xianchao and He, Jian and Zhang, Liping},
  booktitle={Proceedings of the ACM Symposium on Cloud Computing},
  pages={639--653},
  year={2021}
}

## Brief Introduction

<!-- 通过三五句话描述这篇文章，包括 1. 论文的应用场景；2. 论文克服已有方法的局限性；3. 论文主要的技术手段； 4. 论文的预期结果 -->
在云上的深度学习模型推理服务集群中，针对单一容器的资源配置预测。

每个推理模型独占一个容器，为容器分配合理的资源能够节省开销，最大化RPS。先前的方法有使用历史数据推算的，有使用黑盒、白盒、以及基于相似度方法进行搜索的。他们都忽视了推理模型本身在针对各类配置上具有一般性的变化趋势。本文通过使用元学习加few-shot learning的方法，并配合改进后的SMBO方法进行搜索。元学习模型能够提供在各类配置的变化上的一般性趋势信息，并通过few-shot learning能够快速适配到一个新的回归任务，与SMBO方法共同作用，快速搜索到一个近似最优的配置结果。

## Key Methodology

<!-- 分点写，论述论文中主要技术手段的实施过程 -->
- 容器的配置的重要性
  
  - CPU核心数：
    
    CPU 核数在inference serving上会造成性能瓶颈，这是因为inference通常关注QPS，这会有大量的网络带宽需求。所以对于频繁的数据处理和IO需要更高的并行度。

  - GPU显存：

    显而易见，inference任务消耗显存的方式：存放模型的静态占用，动态的中间结果缓存。

  - GPU time-share：

    能够通过CUDA API限制一个容器使用的CUDA SMs的数量，从而隔离共同使用同一个GPU的容器。（大概就是将一个GPU分割成多份）

  - GPU类型：

    显而易见

  - 运行时参数：Batch Size：

    恰当的Batch Size能够提升并行度，减少CPU到GPU的拷贝次数，分期偿还RPC调用的overhead。但是过大的Batch Size会导致GPU显存不足，造成性能下降，也会造成某些请求的等待。

  以上这些参数对于任何一种模型的性能都存在一种**类似**的性能变化趋势。虽然每个曲线的拐点和规模不同，但是它们的趋势类似。本系统深刻地利用了这一点。

- 先前方法的问题

    - 基于历史数据的

        历史数据只有一小部分配置的组合。搜索空间小。并且新的服务没有历史数据。

    - 基于搜索的：

        - 基于黑盒搜索：

            即SMBO，热门的如贝叶斯优化。但是贝叶斯优化对于一个大的、高维的搜索空间的成本很高。并且，它的性能严重依赖于初始抽样。

        - 基于白盒搜索：

            利用一小部分样本，建立一个回归模型。利用先验知识来决定性能会如何变化。
            但是在这里，高维的配置-性能平面过于复杂，很难用少量样本fit。

        - 基于相似度比较：

            利用一个在过去的学习好的benchmarks，随后将新的workload与过去的进行1对1比较相似度。挑选相似度最高的，选择该过去的benchmark使用的配置即可。

            这个方法主要关注1对1的相似度，却没有考虑这些workloads固有的，共有的性能趋势。

            并且这种方法需要更大的一片搜索空间。

- 模型无感知的元模型训练

    考虑模型的性能针对各种配置具有类似的变化趋势，所以想到使用元学习解决这个问题。
    
    在线下训练一个meta-model，能够捕获到普遍的“配置-性能”趋势。
    
    首先定义回归任务$T_i$，该任务为拟合一个真实的训练好的模型的性能的函数。定义该函数为$f_{\theta_i}(x)$，其中$\theta_i$表示从meta-model调优适配到$T_i$的模型。并定义了一个loss函数为真实性能与预测性能的MSE。那么将一个meta-model适配到一个特定的模型需要使用K个真实数据，做随机梯度下降。这就是所谓的few-shot learning。我们的目的就是把meta-model，训练成在K个数据内能将meta-model收敛到该新模型的meta-model。

    具体而言，它遍历所有的训练用的回归任务（每个包含一个真实的模型），首先从这个模型中抽样K个数据，用这K个数据，对现有的meta-model做梯度下降（这就是few-shot learning），得到了一个$\theta_i$模型，作为当前的对回归任务i的模型。遍历完所有的回归任务后，使用所有的$\theta_i$再使用loss函数，并求和。这个和作为训练$\theta_m$要优化到最小的目标函数，这时对$\theta_m$做一次梯度下降优化。这样就训练出来了元模型。

    训练出来的元模型一定能够快速收敛，因为训练它的目标就是找到对于任务的变更更加敏感的元模型。

- 使用SMBO配合元模型进行搜索

    SMBO是一个配置调优的常见方法。它首先随机的初始化一个回归模型，然后迭代地fit模型并且使用它来决定下面是拓展哪个配置。这里就需要在exploration和exploitation之间做权衡，通常使用acquisition函数来指导。

    与一般的SMBO算法不同，这里使用训练好的meta-model作为初始的回归模型，并迭代地将它适配到新的推理模型上。设置K为抽样的预算，即能够进行profile的次数。

    在K次迭代中，每一次首先将现有的模型（一开始为meta-model）进行随机梯度下降更新（使用上一次挑选的数据点），然后对所有可选的配置遍历，计算他们的Acq函数值，选择能使该函数值最大的配置$x_i$，并使用它作为新的采样点，在设备上对它进行evaluation，并把这个点放入新采样的数据集中。以上描述了一个SMBO算法的过程，这里的Acq函数是关键，即如何选择下一个采样点$x_i$。

    需要在exploration和exploitation之间做权衡，这需要对预测的结果有一个置信值。但是调优的回归模型并不能直接得到预测置信值，所以另外定义了置信值：随机梯度下降更新前与更新后的模型对该样本的预测性能的差的绝对值，这等于$| f_{\theta_i} - f_{\theta_i^\prime} |$。这个差值的绝对值越大，说明预测越不准确，那么文中定义Acq函数为：$f_{\theta_i} (x) + \delta Conf$，Conf代表前面的差的绝对值。前面一部分代表这个$x$有多准确，后面代表有多不准确。加起来作为Acq函数的值。这就体现了exploitation和exploration的权衡。



## Data sets & Experimental Design

<!-- 撰写实验环境的设置，实验的对象，实验的比较方面，以及实验的结果（不要列举数据，要概括谈） -->
- 实验环境，设置，对象

    在AWS EC2上做小型实验，在阿里的生产集群上做大型实验。
    
    依据MLPerf Inference benchmark在42个开源模型上做测试。

    对比对象：BO，Ernest，Vizier，Fine-Tuning，Random

- 比较方面

    - 搜索出的配置的性能
    - 搜索本身的开销


## Conclusion And Future Work

<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->
在超参数搜索，配置搜索等领域，使用SMBO是一个较为通常的方法。本文使用了元学习的方法，训练一个meta-model，能够捕获较为普遍的趋势信息，能够经过少量迭代快速适配到具体问题。在本文中，meta-model作为SMBO中的回归模型，解决了BO方法不适用于高维空间，以及它严重依赖于初始抽样的问题。这种元学习的方法在其他领域可能也能起到类似的作用，值得借鉴。