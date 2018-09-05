# 目录 #

- [第一节 Spark的算子](#1)
- [第二节 Spark RDD的高级算子](#2)
- [第三节 Spark编程案例](#3)

***

<h4 id='1'>第一节 Spark的算子</h4>

1. RDD基础
2. Transformation算子
3. Action算子
4. RDD的缓存机制
5. RDD的Checkpoint(检查点)机制:容错机制
6. RDD的依赖关系和Spark任务中的Stage

---

RDD(Resilient Distributed DataSet 弹性分布式数据集)
- 是Spark中最基本也是最重要的数据抽象
- 代表一个不可变、可分区、里面的元素可并行计算的集合
- 具有数据流模型的特点：自动容错、位置感知性调度和可伸缩性
- 允许用户在执行多个查询时显性地将工作集缓存在内存中，后续的查询能够重用工作集，极大地提升了查询速度
- RDD的属性
    - A list of partitions
        - RDD由分区组成
        - 一份分区运行在一个Worker节点上
        - 一个Worker上可以运行多个分区
    - A function for computing each split
        - RDD算子（方法、函数），用于计算数据
    - A list of dependencies on other RDDs
        - RDD存在依赖关系
    - Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned)
    - Optionally, a list of preferred locations to compute each split on (e.g. block locations for an HDFS file)
- 创建RDD
    - val rdd1 = sc.parallelize(Array(1,2,3,4,5), 2)
        - 2:两个分区
        - rdd1.partitions.length
    - 直接读取外部数据源来创建RDD
        - val rdd2 = sc.textFile("hdfs://hadoop001:9000/test.txt")
- 使用RDD的算子对数据进行计算
    - 算子：即函数、方法
    - Transformation算子
        - 不会触发计算，延时加载（计算）
    - Action算子
        - 触发计算，把计算任务提交到Spark集群

RDD算子
- Transformation
    - map(func)：对原来的RDD的每个与元素，进行func运算，返回一个新的RDD
        - 例：map(word=>(word,1))
    - filter：过滤，选择满足条件的元素
    - flatMap：flatten + map
    - mapPartitions(func)：对原来的RDD的每个分区进行func运算，返回一个新的RDD
    - mapPartitionsWithIndex(func)：对原来的RDD的每个分区进行func运算，返回一个新的RDD，并带有分区的下标
    - union：并集
    - intersection：交集
    - distinct：去重
    - groupByKey：分组
    - reduceByKey：分组，会有一个本地操作（相当于MR中的Combiner），效率更高
- Action
    - collect：触发计算
    - count：求数量，类似select count(*) from ...
    - first：取第一个元素
    - take(n)：取RDD中的前n个元素
    - saveAsTextFile：保存数据导文件上，也会触发计算
    - foreach(func)：对每个元素进行func操作
        - 与map区别：没有返回的值

常用的RDD算子示例
```
// 创建一个数值类型的RDD
val rdd1 = sc.parallelize(Array(5,6,7,10,3,100,1,200,13))
// 对每个元素乘以2，并且排序
val rdd2 = rdd1.map(_*2).sortBy(x=>x,true) // true-升序/false-降序
// 过滤filter，选择大于50的元素
val rdd3 = rdd2.filter(_>50)
// 创建字符串类型的RDD
val rdd4 = sc.parallelize(Array("a b c","d e f","x y z"))
// flatMap
val rdd5 = rdd4.flatMap(_.split(" "))
// 集合运算
val rdd6 = sc.parallelize(List(1,2,3,4,5,6,7))
val rdd7 = sc.parallelize(List(6,7,8,9,10))
val rdd8 = rdd6.union(rdd7) // 并集
val rdd9 = rdd6.intersection(rdd7) // 交集
// 分组操作(key,value)
val rdd10 = sc.parallelize(List(("Tom", 100), ("Mary", 200), ("Mike", 50)))
val rdd11 = sc.parallelize(List(("Tom", 200), ("Jerry", 300)))
val rdd12 = rdd10 union rdd11
val rdd13 = rdd12.groupByKey
// rdd13: Array[(String, Iterable[Int])] = Array((Tom,CompactBuffer(100, 200)), (Jerry,CompactBuffer(300)), (Mike,CompactBuffer(50)), (Mary,CompactBuffer(200)))
```

RDD的缓存机制
- 通过persist或cache方法实现缓存
- 需要触发action以后才会执行缓存
```
  /**
   * Persist this RDD with the default storage level (`MEMORY_ONLY`).
   */
  def persist(): this.type = persist(StorageLevel.MEMORY_ONLY)

  /**
   * Persist this RDD with the default storage level (`MEMORY_ONLY`).
   */
  def cache(): this.type = persist()
```
- 默认缓存的位置：MEMORY_ONLY
- 缓存的位置由StorageLevel来定义
```
val rdd1 = sc.textFile("hdfs://hadoop001:9000/test.txt")
rdd1.count // 触发计算，没有缓存

rdd1.cache // 缓存数据，需要触发一个action以后执行
rdd1.count // 触发计算，缓存数据

rdd1.count // 从缓存中计算，速度很快
```

RDD的检查点（checkpoint）
- 辅助RDD的lineage（血统）进行容错的管理
    - 容错机制，当程序出错时，会从最近的检查点的RDD开始往后重做lineage，减少开销
- 检查点分类
    - 本地目录检查点
        - 用于开发和测试环境
        - spark-shell本地模式
        ```
        sc.setCheckpointDir("/...")
        // do something...
        ```
    - HDFS目录检查点
        - 用于生产环境
        - spark-shell集群模式
        ```
        sc.setCheckpointDir("hdfs://hadoop001:9000/...")
        // do something...
        ```

RDD的依赖关系
- RDD和它依赖的父RDD(s)的关系有两种不同类型
    - 窄依赖（narrow dependency）
        - 每一个父RDD的Partition最多被子RDD的一个Partition使用
        - 例：map、union等操作
    - 宽依赖（wide dependency）
        - 多个子RDD的Partition会依赖同一个父RDD的Partition
        - 例：groupBy、join等操作
        - 宽依赖是划分Stage的依据

任务的阶段（Stage）
- 原始的RDD通过一系列的转换形成了DAG
    - DAG（Directed Acyclic Graph）有向无环图
- 根据RDD之间的依赖关系，将DAG划分成不同的Stage
    - 窄依赖：partition的转换处理在Stage中完成计算
    - 宽依赖：有Shuffle存在，只能在父RDD处理完成后，才能开始接下来的计算
        - 因此，宽依赖是划分Stage的依据

***

<h4 id='1'>第二节 Spark RDD的高级算子</h4>

1. mapPartitionsWithIndex
2. aggregate
3. aggregateByKey
4. coalesce与repartition
5. 其他高级算子

---

mapPartitionsWithIndex
- 作用：把RDD中的每个分区和对应的分区号取出来进行相应的计算
```
def mapPartitionsWithIndex[U](
    f: (Int, Iterator[T]) ⇒ Iterator[U], 
    preservesPartitioning: Boolean = false)
    (implicit arg0: ClassTag[U]): RDD[U]
```
- 参数f: (Int, Iterator[T]) ⇒ Iterator[U]
    - Int：分区号
        - mapPartitions无这个参数
    - Iterator[T]：该分区中的元素
    - => Iterator[U]：对该分区中的元素进行运算（f）的结果
```
scala> var rdd1 = sc.parallelize(List(1,2,3,4,5,6,7),2)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[25] at parallelize at <console>:24

scala> def func1(index:Int,iter:Iterator[Int]):Iterator[String]={
     | iter.toList.map(x=>"[PartitionID:" + index + ",value:" + x + "]").iterator
     | }
func1: (index: Int, iter: Iterator[Int])Iterator[String]

scala> rdd1.mapPartitionsWithIndex(func1).collect
res0: Array[String] = Array(
    [PartitionID:0,value:1], 
    [PartitionID:0,value:2], 
    [PartitionID:0,value:3], 
    [PartitionID:1,value:4], 
    [PartitionID:1,value:5], 
    [PartitionID:1,value:6], 
    [PartitionID:1,value:7])
```

aggregate
- 聚合操作
```
def
aggregate[U](zeroValue: U)(seqOp: (U, T) ⇒ U, combOp: (U, U) ⇒ U)(implicit arg0: ClassTag[U]): U
```
- zeroValue：执行聚合操作的初始值
- seqOp：局部的聚合操作
- combOp：全局的聚合操作
- 例1：每个分区中的最大值求和
    ```
    val rdd2 = sc.parallelize(List(1,2,3,4,5),2)
    rdd2.aggregate(0)(math.max(_,_),_+_)    --> 7
    rdd2.aggregate(10)(math.max(_,_),_+_)    --> 30
    ```
    - zeroValue=0
    - seqOp=math.max(\_,_)
        - 局部聚合处理在分区上，取最大值
    - combOp=\_+_
        - 全局聚合处理在所有分区，求和
    - 初始值为10时：zeroValue+max(zeroValue,1,2)+max(zeroValue,3,4,5)=30
- 例2：对局部求和，并对全局求和
    ```
    rdd2.aggregate(0)(_+_,_+_)    --> 15
    rdd2.aggregate(10)(_+_,_+_)    --> 45
    ```
    - zeroValue+(zeroValue+1+2)+(zeroValue+3+4+5)
- 例3：字符串
    ```
    scala> val rdd2 = sc.parallelize(List("a","b","c","d","e","f"),2)
    rdd2: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[29] at parallelize at <console>:24

    scala> def func2(index:Int,iter:Iterator[String]):Iterator[String]={
        | iter.toList.map(x=>"[ID:"+index+",val:"+x+"]").iterator}
    func2: (index: Int, iter: Iterator[String])Iterator[String]

    scala> rdd2.aggregate("")(_+_,_+_)
    res0: String = abcdef
    scala> rdd2.aggregate("|")(_+_,_+_)
    res1: String = ||abc|def
    ```
- 处理结果可能会有不同，取决于哪个分区的局部聚合先处理完成

aggregateByKey
- 和aggregate类似，先局部聚合，再全局聚合
- 区别在于该算子处理的是kv数据类型
- 例：
    ```
    val pairRDD = sc.parallelize(List(("cat",2),("cat",5),("mouse",4),("cat",12),("dog",12),("mouse",2)),2)

    def func3(index:Int,iter:Iterator[(String,Int)]):Iterator[String] = {
    iter.toList.map(x=>"[ID:"+index+",value:"+x+"]").iterator
    }

    pairRDD.mapPartitionsWithIndex(func3).collect
    --> Array[String] = Array(
        [ID:0,value:(cat,2)], 
        [ID:0,value:(cat,5)], 
        [ID:0,value:(mouse,4)], 
        [ID:1,value:(cat,12)], 
        [ID:1,value:(dog,12)], 
        [ID:1,value:(mouse,2)])

    pairRDD.aggregateByKey(0)(math.max(_,_),_+_).collect
    --> Array[(String, Int)] = Array((dog,12), (cat,17), (mouse,6))

    pairRDD.aggregateByKey(0)(_+_,_+_).collect
    --> Array[(String, Int)] = Array((dog,12), (cat,19), (mouse,6))

    pairRDD.reduceByKey(_+_).collect
    --> Array[(String, Int)] = Array((dog,12), (cat,19), (mouse,6))
    ```

coalesce与repartition
- 都是对RDD中的分区进行重分区
```
def
coalesce(numPartitions: Int, shuffle: Boolean = false, partitionCoalescer: Option[PartitionCoalescer] = Option.empty)(implicit ord: Ordering[T] = null): RDD[T]

def
repartition(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T]
```
- coalesce：默认不会进行shuffle(false)
- repartition：会进行shuffle，将数据通过网络进行重分区
```
val rdd4 = sc.parallelize(List(1,2,3,4,5,6,7,8,9), 2)
// 分区0:1,2,3,4
// 分区1:5,6,7,8,9

val rdd41 = rdd4.coalesce(3)
rdd41.partitions.length
--> Array[String] = Array(
    [PartitionID:0,value:1], 
    [PartitionID:0,value:2], 
    [PartitionID:0,value:3], 
    [PartitionID:0,value:4], 
    [PartitionID:1,value:5], 
    [PartitionID:1,value:6], 
    [PartitionID:1,value:7], 
    [PartitionID:1,value:8], 
    [PartitionID:1,value:9])

val rdd42 = rdd4.repartition(3)
rdd42.partitions.length
--> Array[String] = Array(
    [PartitionID:0,value:3], 
    [PartitionID:0,value:7], 
    [PartitionID:1,value:1], 
    [PartitionID:1,value:4], 
    [PartitionID:1,value:5], 
    [PartitionID:1,value:8], 
    [PartitionID:2,value:2], 
    [PartitionID:2,value:6], 
    [PartitionID:2,value:9])

val rdd43 = rdd4.coalesce(3, true)
rdd43.partitions.length
--> Array[String] = Array(
    [PartitionID:0,value:3], 
    [PartitionID:0,value:7], 
    [PartitionID:1,value:1], 
    [PartitionID:1,value:4], 
    [PartitionID:1,value:5], 
    [PartitionID:1,value:8], 
    [PartitionID:2,value:2], 
    [PartitionID:2,value:6], 
    [PartitionID:2,value:9])
```

***

<h4 id='1'>第三节 Spark编程案例</h4>