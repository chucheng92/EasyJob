## Spark

本文环境：Hadoop2.7 Spark2.1.1

### Spark知识点

简介:快速通用的集群计算平台

生态和组件:Spark Core、Spark SQL、Spark Streaming、Mlib、Graphx、Cluster Managers

Spark和Hadoop的比较：

Hadoop应用场景，离线处理，对时效性要求不高。

Spark应用场景，时效性要求高，机器学习。

其他：Spark是紧密集成的。

### Spark安装

依赖 Spark2.0以上需要Scala2.11

### RDD

RDD(Resilient Distributed Datasets),弹性分布式数据集

常用方法：parallelize，foreach

RDD创建方法：
1、已存在的集合传递给SparkContext的parallelize方法，测试用
2、textFile加载

### RDD Transformation

Map、FlatMap、Filter、distinct、union、intersection、subtract

### RDD Action

在RDD上计算出一个结果，把结果返回给driver program或保存在文件系统

reduce、count、save、collect、foreach

reduce：接收一个函数，作用在RDD两个类型相同的元素上，返回新元素

collect：遍历rdd内容，向driver program返回rdd的内容（测试用）
大数据集时用saveAsTextFile保存

take（n）：返回RDD的n个元素

top（n）：根据默认的比较器返回top

foreach：计算RDD的每个元素，但不返回本地，配合print友好打印数据

### RDD的特性

1、血统关系图
Spark维护RDD之间的依赖关系和创建关系，称作血统关系图
Spark使用血统关系图来就计算每个RDD的需求和恢复丢失的数据

2、延迟计算
Spark对RDD的计算是在第一次使用action操作的时候

3、RDD.persist()

默认每次在RDD上进行action操作时，spark都重新计算RDD。
如果想重复使用一个RDD，可以使用RDD.persist()
unpersist方法从缓存移除

|      级别           | 空间占用 | cpu消耗 | 内存 | 硬盘 |
| ------------------- | -----  | ------- | --- | --- |
| MEMORY_ONLY         | High   | Low    |  Y   |  N  | 
| MEMORY_ONLY_SEQ     | Low    | High   |  Y   |  N   |
| DISK_ONLY           | Low    | High   |  N   |  Y   |
| MEMORY_AND_DISK     | High   | Medium | Some | Some |
| MEMORY_AND_DISK_SEQ | Low    | High   | Some | Some |


### KeyValue对RDD

创建：如map函数
Transformation：reduceByKey（相同key结合）、groupByKey（相同key的value分组）、combineByKey

mapValues：函数作用于pairRDD的每个元素，对value操作，key不变
flatMapValues：符号化的时候使用，也是对value操作
keys：返回keys
values：返回values
sortByKey()：按照key排序的RDD

combineByKey(): 参数createCombiner、mergeValue、mergeCombiners、partitioner，最常用的基于key的聚合函数，返回的类型可以与输入类型不一样

如果是新元素，使用createCombiner，如果是已存在的key，使用mergeValue
合计每个partition的结果的时候，使用mergeCombiners函数

**详解combineByKey**

def combineByKey[C](createCombiner: (V) => C, mergeValue: (C, V) => C, mergeCombiners: (C, C) => C): RDD[(K, C)]

def combineByKey[C](createCombiner: (V) => C, mergeValue: (C, V) => C, mergeCombiners: (C, C) => C, numPartitions: Int): RDD[(K, C)]

def combineByKey[C](createCombiner: (V) => C, mergeValue: (C, V) => C, mergeCombiners: (C, C) => C, partitioner: Partitioner, mapSideCombine: Boolean = true, serializer: Serializer = null): RDD[(K, C)]

其中的参数：

createCombiner：组合器函数，用于将V类型转换成C类型，输入参数为RDD[K,V]中的V,输出为C

mergeValue：合并值函数，将一个C类型和一个V类型值合并成一个C类型，输入参数为(C,V)，输出为C

mergeCombiners：合并组合器函数，用于将两个C类型值合并成一个C类型，输入参数为(C,C)，输出为C

numPartitions：结果RDD分区数，默认保持原有的分区数

partitioner：分区函数,默认为HashPartitioner

mapSideCombine：是否需要在Map端进行combine操作，类似于MapReduce中的combine，默认为true


举例理解：

假设我们要将一堆的各类水果给榨果汁，并且要求果汁只能是纯的，不能有其他品种的水果。那么我们需要一下几步：

1 定义我们需要什么样的果汁。

2 定义一个榨果汁机，即给定水果，就能给出我们定义的果汁。--相当于hadoop中的local combiner

3 定义一个果汁混合器，即能将相同类型的水果果汁给混合起来。--相当于全局进行combiner


那么对比上述三步，combineByKey的三个函数也就是这三个功能

1 createCombiner就是定义了v如何转换为c

2 mergeValue 就是定义了如何给定一个V将其与原来的C合并成新的C

3 就是定义了如何将相同key下的C给合并成一个C

var rdd1 = sc.makeRDD(Array(("A",1),("A",2),("B",1),("B",2),("C",1)))

rdd1.combineByKey(
(v : Int) => List(v),　　　　　　　　　　　　　--将1 转换成 list(1)
(c : List[Int], v : Int) => v :: c,　　　　　　　--将list(1)和2进行组合从而转换成list(1,2)
(c1 : List[Int], c2 : List[Int]) => c1 ::: c2　　--将全局相同的key的value进行组合
).collect
res65: Array[(String, List[Int])] = Array((A,List(2, 1)), (B,List(2, 1)), (C,List(1)))