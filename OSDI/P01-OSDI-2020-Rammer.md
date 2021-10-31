# Title

<!-- 此部分是论文标题-->
Rammer: Enabling Holistic Deep Learning Compiler Optimizations with rTasks

## Website
<!-- 网址，有DOI的建议用DOI地址-->
https://www.usenix.org/conference/osdi20/presentation/ma

## Citing

<!-- 引用格式，建议使用latex格式-->
@inproceedings{ma2020rammer,
  title={Rammer: Enabling Holistic Deep Learning Compiler Optimizations with rTasks},
  author={Ma, Lingxiao and Xie, Zhiqiang and Yang, Zhi and Xue, Jilong and Miao, Youshan and Cui, Wei and Hu, Wenxiang and Yang, Fan and Zhang, Lintao and Zhou, Lidong},
  booktitle={14th $\{$USENIX$\}$ Symposium on Operating Systems Design and Implementation ($\{$OSDI$\}$ 20)},
  pages={881--897},
  year={2020}
}

## Brief Introduction

<!-- 通过三五句话描述这篇文章，包括 1. 论文的应用场景；2. 论文克服已有方法的局限性；3. 论文主要的技术手段； 4. 论文的预期结果 -->
本文针对深度学习编译器领域。深度学习模型使用DFG（数据流图）表示，它自然包含两种并行度：算子间并行和算子内部计算的并行。已有方法采用两层调度：高层框架每次为底层提供一批可并行的算子，底层框架接受这些算子，再动态地在硬件上进行并行执行的调度。这种方式在高层为底层提供算子时存在overhead，并且忽视了算子内与算子间两层并行之间的微妙关系。Rammer重新定义了算子，将其内部的最小可执行单元暴露出来，并设计了能够抽象不同底层硬件，并提供同步和调度原语的虚拟设备，最后用编译的方法将调度决策从运行时转为编译时，编译同时考虑了算子内与算子间的并行。编译出针对虚拟设备的IR，再最后转化为目标设备的机器码。

## Key Methodology

<!-- 分点写，论述论文中主要技术手段的实施过程 -->
1. 重新定义DFG中的算子

    在Rammer中的DFG，其中每个算子被称为rOperator，这种rOperator包含一系列独立、同质的rTask，一个rTask是指能够直接在一个并行计算单元运行的一个最小的计算单元。举例说明：矩阵乘法算子中，rTask为每个tile的计算。这里每个tile的计算都是同质的。这就正好映射到了GPU中的SIMD架构，同一条指令能够计算多个操作数。所以这里的rTask必须是同质的，才能使用相同的指令。
    
    在给定一个rOpeartor下，Rammer依赖如TVM这样的kernel生成器，来生成一系列rTask。所以rOperator能够有不同的实现版本，每个rOperator的实现叫rKernel。

2. 虚拟并行设备
   
   加速器一般不提供直接映射rTask到一个指定的计算单元的接口。为解决这个挑战，Rammer将硬件加速器抽象成一种软件管理的虚拟设备（vDevice），并抽象里面的每个虚拟计算单元（vEU），每个都可以独立执行rTask。

   Rammer将rTask-aware的DFG经过编译后，组织成在vDevice上执行的多段rProgram，rProgram由二维数组组织，第一维为vEU的id，第二维为在这个计算单元上进行的rTask id。将DFG编译成多段而不是一段rProgram是因为，rProgram是对底层设备的一次调用，而如果把全部计算放在一次调用里效率较差，或者不可能做到。
   
   为保证rTask之间执行顺序，提供了像是内存屏障一样的barrier-rTask，用于等待某些rTask执行结束。

   最终，这些rProgram作为到底层硬件之间的IR，被转化成具体的硬件代码。

3. rTask-aware DFG的编译器

    编译器接受一个rTask-aware的DFG，目标是生成在vDevice上的rProgram IR。它将调度机制与调度策略解耦合。在机制方面，它提供两种调度接口用于生成执行计划，另外提供profiler用来提供rTask和rProgram的执行信息，用于指导调度计划的生成。

    - 调度接口
  
        调度接口包括Append和Wait，Append就是将某个rOperator中的rTask以线性执行的顺序映射到一个特定的vEU上，再添加到执行计划中。

        Wait则代表等待某些rTask的执行结束，隐式地添加一个barrier-rTask到执行计划当中。

    - 调度策略
  
        在DFG中，使用BFS找出一批在最边缘的算子（这些算子组成一个wave），这些算子能够在算子间并行，对这批算子做一次调度。

        首先对这批rOperator，要选择它们的rKernel实现。如何选择？这里采取了启发式方法：对于每个rOperator的不同rKernel实现，有最“快速”和“效率”的两个版本。“快速”是指执行时间最短，“效率”是指rTask的数量乘以执行时间的积最小。那么如果在这个wave中的所有rOperator都使用最快的实现，并且将这些实现中的全部rTask都不能完全占用满全部的并行计算单元，就选择它们。否则，将会找到最高效的rKernels，并执行一次profiling，如果这些rKernel的执行时间更短，就选择它们，否则还是选择最快的rKernel。

        随后将对所有rKernel中的所有rTask分配它的vEU。由于在选择rKernel时，做了profiling，知道每个rTask的执行时间，并且已知当前已经生成的部分rProgram，所以能够知道在当前的rProgram的执行下，哪个vEU能够最早的空闲下来。选择能够最早执行这个rTask的vEU即可。在这一步中，编译器能够将不同算子间的rTask共同考虑。

4. 目标设备代码生成
   
   主要讲了CUDA的rOperator如何实现，如何绑定vEU到实际的CUDA streaming multiprocessor。也讲了到AMD GPU和IPU、x86 CPU的实现。

   对CUDA和底层GPU架构不太了解，目前看不太懂。



## Data sets & Experimental Design

<!-- 撰写实验环境的设置，实验的对象，实验的比较方面，以及实验的结果（不要列举数据，要概括谈） -->
- 比较对象：

    DNN框架：TensorFlow

    DNN编译器：TVM，TensorFlow-XLA，RammerBase（为比较Rammer内部的特性，使用Rammer的基础代码，利用现有的DNN编译技术，写了一个二层调度的编译器。）

    对NVIDIA GPU深度优化的推理库：TensorRT

- 使用模型：

    ResNeXt，NASNet，AlexNet，LSTM-TC，Deep-Speech2，Seq2Seq。

- 数据集：
    
    CIFAR-10，ImageNet，LibriSpeech，synthetic datasets

- 在CUDA GPU上测试

    - 端到端的整体性能
      - 不同batch size
      - 不同input size
    - GPU利用率
    - 调度overhead
    - 算子间与算子内调度的相互作用测算（与RammerBase共同比较）

        分别对比：总是选择最快的rKernel，以及使用Rammer的调度策略选择的rKernel。

    - barrier-rKernel的同步overhead消耗时间测算

- 在其他加速器上的测试
  - 在ROCm GPU上的测试
    - 端到端整体性能
  - 在Graphcore IPU上的测试




## Conclusion And Future Work

<!-- 作者或者阅读者对本文工作的总结，以及未来可能的改进方向 -->
1. 提供rTask-operation的抽象，将细粒度的算子内并行暴露出来。
2. 将带有并行计算单元的现代加速器进行虚拟化的抽象，将硬件细粒度的调度能力暴露出来。
3. 利用DNN的计算稳定性，来将运行时调度转化为生成编译期的执行计划。