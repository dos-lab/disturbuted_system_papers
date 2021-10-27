# 论文阅读

论文阅读，读书笔记。

## 泛读论文目录

to do.

## 精读论文目录

- P-系统性能保障
  - 01.编译优化
    - 云计算系统：
    - 深度学习系统：[Baechi@SoCC20](SoCC/P01-SoCC-2020-Baechi.md)
    - 服务器计算/函数系统：[Ray@OSDI18](OSDI/P01-OSDI-1028-ray.md) 
  - 02.虚_拟_化
    - 深度学习系统：
  - 03.容量规划
    - 大数据系统：[CherryPick@NSDI17](NSDI/P03-NSDI-2017-CherryPick.md), [PARIS@SoCC17](SoCC/P03-SoCC-2017-PARIS.md)
    - 深度学习系统：[HeterBO@IPDPS20](IPDPS/P03-IPDPS-2020-HeterBO.md)
  - 04.调度优化  
    - 大数据系统： [BSBSched@HPDC19](HPDC/P04-HPDC-2019-BSBSched.md), [CR@HPDC19](HPDC/P04-HPDC-2019-CR.md), [Elasecutor@SoCC18](SoCC/P04-SoCC-2018-Elasecutor.md)
    - 无服务器系统：
    - 深度学习系统：[BytePS@OSDI20](OSDI/P04-OSDI-2020-bytePS.md), [Gavel@OSDI20](OSDI/P04-OSDI-2020-gavel.md), [Gandiva@OSDI20](OSDI/P04-OSDI-2818-Gandiva.md), [Tiresias@NSDI19](NSDI/P04-NSDI-2019-Itresias.md), [ModelGoverance@ATC18](ATC/P04-ATC-2018-MG.md), [Optimus@EuroSys18](EuroSys/P04-EuroSys-2018-Optimus.md), [TicATac@MLSys18](MLSys/R01-MLSys-2018-TICTAC.md), [Continuum@SoCC18](SoCC/P04-SoCC-2018-Continuum.md)
    - 区块链系统：
    - 存储系统：
  - 05.性能预测
    - 深度学习系统： [clcokwork@OSDI20](OSDI/P05-OSDI-2020-Clockwork.md)
    - 大数据系统：[PA@NSDI18](NSDI/P05-NSDI-2018-PA.md) 
  - 06.基准测试
    - 云计算系统：
    - 区块链系统：
    - 深度学习系统：
    

# 撰写格式

需要对不同论文的发表时间、主要领域、会议名称和成果名称进行明确标注，并在笔记名中体现，具体格式和选项如下：

1. 精度论文阅读笔记的文件名，必须以Markdown为格式，其命名规范是：论文分类编号-论文会议缩写-论文年份-论文名称（缩写）.md
2. 泛读论文阅读笔记的文件名，必须以markdown为格式，其命名规范是：论文分类-泛读.md

根据AS组相关的研究方向和综述的相关工作，论文分类以下表为准，结合自己研究研究方向可适当扩充：

|  分类编号   |  分类名称   | 主要撰写人员  | 次要阅读人员  | 系统研制 |
|  ----   |  ----  | ----  | ----  | ----  |
|  P01    | 编译优化  | 许源佳 | 吴恒 |      |
|  P02    | 虚_拟_化  | 罗荣周 | 吴恒 | [kube-gpu](https://github.com/kubesys/kube-gpu)       |
|  P03    | 容量规划  | 胡艺   | 吴恒、吴悦文 |         |
|  P04    | 调度优化  | 杨晨、曾浩、高赫然、施建峰 | 吴恒、吴悦文、许源佳 |        |
|  P05    | 性能预测  | 杨晨、胡艺 | 吴恒、吴悦文 |        |
|  P06    | 基准测试  | 余甜、施建峰| 吴恒 |        |


3. 精度论文按照不同会议的文件夹进行划分，各个文件夹相互独立，避免整理困难，主要有低年级博士生和高年级硕士生更新维护；
4. 泛读文件在根目录，一般由高年级博士生维护；

5. 精度论文的笔记模板是template.md，其内有详细撰写内容要求，低年级博士生请每周更新2篇论文笔记，放在对应会议的文件夹下，周一时候放出来由大家讨论。
6. 目前已经提交的论文主要由许源佳整理，后续希望每个人都能把自己的阅读论文的笔记放出来，事情的推进，这个时代下不是一个人能搞定的。
7. 本项目所有一级目录均为文件或者期刊名称

# 论文阅读范围

不要尝试在google scholar搜索关键字，所有系统论文相关的最新成果，均在CCF B类以上会议和MLSys会议中完全涵盖（顶级期刊一般是这些工作的延伸），只需要从这些文章中选择确定研究方向，并推进自己的工作：

参考链接： https://www.ccf.org.cn/Academic_Evaluation/By_category/

注意，一般关注的领域是：（计算机体系结构/并行与分布计算/存储系统）、（软件工程/系统软件/程序设计语言）这两个；

少部分论文可能关注（人工智能）和（计算机网络），其他均不是团队关注的重心。
