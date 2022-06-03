# 整体流程

以上下文的配置为核心，手工定义每个算子可以调整的配置空间，作为早期的算子调优方式，现有TVM中很多算子均采用手工定义的方式实现优化，相对Ansor适用于更多的异构资源。

在定义即将搜索的配置空间后，初始化任务和搜索器，其中任务针对单个算子进行设置，也可以通过整个模型导出分别针对每个算子的任务；然后指定搜索器的选项，依次根据每个搜索器的空间，分别进行每个算子在不同配置下的编译、运行，并统计运行结果。

# 代码跟踪

以 https://github.com/dos-lab/OpBench/blob/main/test/test_op_autotvm.py 为例进行跟踪：

1. 定义上下文环境，定义新算子矩阵乘法的搜索空间：

```
    ##### define space begin #####
    cfg = autotvm.get_config()
    cfg.define_split("tile_y", y, num_outputs=2)
    cfg.define_split("tile_x", x, num_outputs=2)
    ##### define space end #####
```

2. cfg作为ConfigEntity实例，定义如何扩展搜索空间，核心在类ConfigSpace中，进入TVM源码，查看https://github.com/dos-lab/as-tvm/blob/89061fafa5eff215156f11fccfa258a617466724/python/tvm/autotvm/task/space.py ：查看其中方法，可见当前仅支持切分、重排序、注释和knob四类，作为可以性能调优的基本原语：

```
def define_split
def define_reorder
def define_annotate
def define_knob
```

3. 然后继续回到https://github.com/dos-lab/OpBench/blob/main/test/test_op_autotvm.py 实现对一个算子的性能搜索任务的定义：

```
task = autotvm.task.create("tutorial/matmul", args=(N, L, M, "float32"), target="llvm")
print(task.config_space)
```

跟进create函数：https://github.com/dos-lab/as-tvm/blob/6141cac635fbdaad25b0f8ec3bce130e787922b5/python/tvm/autotvm/task/task.py ， 其核心是对ConfigSpace的定义：

```
ret.config_space = ConfigSpace()
```

可以发现，其又回到了2中的函数，能够回去到对应的配置空间

4. 然后创建了搜索参数配置，主要是指定了编译器和运行器：

```
measure_option = autotvm.measure_option(builder="local", runner=autotvm.LocalRunner(number=5))
```

跟进后可以发现，builder和runner的实例类型分别是：LocalBuilder, LocalRunner，其中builer使用多线程方式进行多线程的刷子编译，runner能够统计实际运行的结果，即是MeasureResult，可以看到对运行结果的字段的含义；

5. 最终，实例化搜索器，并存储真实的性能结果。搜索器有多重实例，这里以随机搜索为例分析其实际执行过程，其核心是执行tuner的init函数和tune函数：

```
tuner = autotvm.tuner.RandomTuner(task)
tuner.tune(
    n_trial=20,
    measure_option=measure_option,
    callbacks=[autotvm.callback.log_to_file("matmul.log")],
)
```

其在评估每个算子的过程中，主要使用批量评估的方式，在https://github.com/dos-lab/as-tvm/blob/29f64c6c5dc5b673987afaba2759f2ef6c085530/python/tvm/autotvm/tuner/tuner.py 中所示：

```
 measure_batch = create_measure_batch(self.task, measure_option)
```

跟进后可以发现，其主要完成build和run的实例化，控制输出最终结果，其中核心是measureinput的输入，即是self.task，其属性中，最重要的是ConfigEntity属性，在https://github.com/dos-lab/as-tvm/blob/89061fafa5eff215156f11fccfa258a617466724/python/tvm/autotvm/task/space.py中实现，完成了对相关配置输入输出的序列化操作，均以json格式为基础，其中，可以发现json的关键字信息代表的含义：


```
        for k, v in self._entity_map.items():
            if isinstance(v, SplitEntity):
                entity_map.append((k, "sp", v.size))
            elif isinstance(v, ReorderEntity):
                entity_map.append((k, "re", v.perm))
            elif isinstance(v, AnnotateEntity):
                entity_map.append((k, "an", v.anns))
            elif isinstance(v, OtherOptionEntity):
                entity_map.append((k, "ot", v.val))
            else:
                raise RuntimeError("Invalid entity instance: " + v)
```

# 反思和分析

可以利用这些优化原语的导出结果和性能表现：
1. 构造不同使用资源情况下的算子性能表现（切片多少，线程使用量等）
2. 比较不同资源上的性能拐点；
3. 分析哪些硬件信息和算子依赖关系可以知道减少算子的性能优化代价，以及提升优化、性能预测的准确度。