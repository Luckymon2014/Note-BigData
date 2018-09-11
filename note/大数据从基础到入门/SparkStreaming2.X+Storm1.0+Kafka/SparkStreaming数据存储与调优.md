# 目录 #

- [第一节 缓存与持久化机制](#1)
- [第二节 Checkpoint机制](#2)
- [第三节 部署、升级和监控应用程序](#3)
- [第四节 容错机制以及事务语义详解](#4)
- [第五节 架构原理剖析](#5)
- [第六节 性能调优](#6)

***

<h4 id='1'>第一节 缓存与持久化机制</h4>

缓存与持久化机制
- 与RDD类似，可以手动控制，将数据流中的数据持久化到内存中
- 共享使用内存中的一份缓存数据
```
DStream.persist() // 默认MEMORY_ONLY_SER
DStream.cache() // 默认调用persist()方法
```
- 对于窗口操作，以及基于状态的操作（updateStateByKey），默认隐式开启了持久化机制，即默认会将DStream中的数据缓存到内存中，不需要手动调用persist()方法
- 对于网络接受数据的输入流（Socket、Kafka、Flume等）,默认持久化级别是将数据复制一份，便于容错，即MEMORY_ONLY_SER_2
- 与RDD不同的是，默认的持久化级别，统一都是要序列化的

***

<h4 id='2'>第二节 Checkpoint机制</h4>

Checkpoint
- 实时计算程序特点：需要7*24运转
- 必须要能够对与应用程序逻辑无关的失败进行容错
- 对于一些将多个batch的数据进行聚合，有状态的transformation操作，非常有用
- 避免由于依赖链越来越长，导致越来越长的失败恢复时间
- 使用了有状态的transformation操作时，必须启用Checkpoint机制

启用Checkpoint机制
- 配置一个文件系统（如HDFS）的目录，来启用checkpoint机制，数据会写入该目录
```
ssc.checkpoint("fs_dir")
```

***

<h4 id='3'>第三节 部署、升级和监控应用程序</h4>

部署应用程序
- 需要由一个集群资源管理器
    - Standalone模式下的Spark集群
    - Yarn模式下的Yarn集群
- 打包应用程序为一个jar包
- 为Executor配置充足的内存
    - 执行窗口长度为10分钟的窗口操作，Executor的内存资源必须足够保存10分钟内的数据
- 配置checkpoint

升级应用程序
1. 升级后的Spark应用程序直接启动，先与旧的并行执行，然后停止旧应用程序
    - 只适用于允许多个客户端读取各自独立的、相同的数据
2. 关闭已经在运行的应用程序，部署升级后的应用程序，启动
    - 只适用于支持数据缓存的数据源

监控应用程序
- 通过Spark Web UI显示相关信息
    - 处理时间：每个batch的数据的处理耗时
    - 调度延迟：一个batch在队列中阻塞住，等待上一个batch完成处理的时间

***

<h4 id='4'>第四节 容错机制以及事务语义详解</h4>

容错机制
- 大多数情况下，数据是通过网络接收的，接收到的数据必须被复制到多个Worker节点上的Executor内存中，默认的复制因子是2
- 出现失败事件时，有两种数据需要被恢复
    - 数据接收到了，并且已经复制过：当一个Worker节点挂掉时，可以继续存活，因为在别的Worker节点上还有一份副本
    - 数据接收到了，但是正在缓存中，等待复制的：因为还没有复制，恢复的唯一办法是重新从数据源获取一份

容错事务语义
- 流式计算系统的容错语义：以一条记录能够被处理多少次来衡量
- 三种类型的语义
    - 最多一次：每条记录可能会被处理一次或不处理，可能有数据丢失
    - 至少一次：每条记录会被处理一次或多次，保证数据零丢失，可能会重复处理
    - 一次且仅一次：每条记录只会被处理一次，没有数据丢失，不会重复处理，最强的一种容错语义

基础容错语义
- 应用系统要求有且仅有一次的语义，每条数据都保证只能接收一次、计算一次、推送一次
- 实现语义的步骤如下
    - 接收数据：不同的数据源提供不同的语义保障
        - 基于文件的数据源
        - 基于Receiver的数据源
            - 可靠的Receiver：接收到数据并且将数据复制之后，对数据源执行确认操作，数据不会丢失
            - 不可靠的Receiver：不会发送确认操作，当Worker节点失败的时候，可能会导致数据丢失
    - 计算数据：所有接收到的数据一定只会被计算一次
    - 推送数据：output操作默认能确保至少一次的语义，但是用户可以实现它们自己的事务机制，来确保一次且仅一次的语义
        - 幂等更新：多次写操作，都是相同的数据
        - 事务更新：所有的操作都应该做出事务的，让写入操作执行一次且仅一次
        ```
        DStream.foreachRDD {
            (rdd, time) => rdd.foreachPartition {
                partitionIterator =>
                val partitionId = TaskContext.get.partitionId()
                val uniqueId = generateUniqueId(time.milliseconds, partitionId) // partitionId和foreachRDD传入的时间，可以构成一个唯一的标识
            }
        }
        ```

***

<h4 id='5'>第五节 架构原理剖析</h4>

架构原理
- Spark Streaming相对其他流处理系统最大的优势，在于流处理引擎和数据处理在同一个软件栈
    - Spark Streaming功能主要包括流处理引擎的流数据接收与存储以及批处理作业的生成与管理
    - Spark Core负责处理Spark Streaming发送过来的作业
- 流数据处理大致分为四步
    - 启动流处理引擎
    - 接收及存储流数据
    - 处理流数据
    - 输出处理结果
- StreamingContext
    - 初始化：创建内部的一些关键组件
    - DStreamGraph
        - 存放DStream以及DStream之间的依赖关系信息
    - JobScheduler
        - ReceiverTracker
            - 数据接收器Receiver的管理者
        - JobGenerator
            - 批处理作业生成器
- Worker
    - Executor
        - Receiver
            - 负责数据接收，将数据保存到Executor相关联的BlockManager中
        - BlockManager
0. StremingContext初始化
1. StreamingContext.start()
    - Spark集群中的某个Worker节点的Executor中启动输入DStream的Receiver
2. Receiver从Kafka、Flume等外部源中获取数据
3. Receiver将数据保存到BlockManager
4. Receiver发送一份数据到StreamingContext的ReceiverTracker中
5. JobGenerator每隔一个batch间隔去ReceiverTracker中获取一次时间间隔内的数据信息，然后将数据创建为一个RDD（每个batch对应一个RDD，也就是DStream中的一个时间段对应一个RDD），然后根据DStreamGraph定义的算子和各个DStream之间的依赖关系生成Job，通过JobScheduler提交Job
6. 每个Spark Streaming底层的小Job的执行流程，与Spark内核架构师一样的

***

<h4 id='6'>第六节 性能调优</h4>

性能调优
- 数据接收并行度调优
    - 创建更多输入DStream和Receiver
    - 通过参数spark.streaming.blockInterval(默认200ms)调节Block interval
- 数据处理并行度调优
    - 通过参数spark.default.parallelism调整默认的并行task数量
    - 使用reduceByKey等算子操作时，传入第二个参数，手动指定操作的并行度
- 数据序列化调优
    - 数据序列化造成的系统开销可以由序列化格式的优化来减小
    - 使用Kryo序列化类库可以减小CPU和内存的性能开销
- batch interval调优
    - 检查每个batch的处理时间延迟
    - 如果处理时间与batch interval基本吻合，那么应用就是稳定的，否则应提升数据处理速度或者增加batch interval
- 内存调优，降低内存使用和GC开销
    - DStream持久化，使用Kryo序列化机制或者对数据进行压缩
        - 通过参数spark.rdd.compress参数控制
    - 清理旧数据，如果需要让Spark保存更长时间的数据，直到SparkSQL查询结束，可以使用streamingContext.remeber()方法来实现
    - CMS垃圾回首器，使用并行的mark-sweep垃圾回收机制，用来保持GC低开销，来减少batch的处理时间