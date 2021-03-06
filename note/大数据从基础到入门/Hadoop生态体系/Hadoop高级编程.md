# 目录 #

- [第一节 MapReduce Shuffle过程](#1)
- [第二节 使用MRUnit进行单元测试过程](#2)
- [第三节 MapReduce 数据压缩Snappy,Gzip,LZO](#3)
- [第四节 MapReduce Partitioner,Combiner实现及应用](#4)
- [第五节 Mapreduce 高级编程](#5)

***

<h4 id='1'>第一节 MapReduce Shuffle过程</h4>

1. 了解MapReduce Shuffle原理
2. 掌握MapReduce Shuffle优化方法

---

MapReduce Shuffle
- 作用：将数据从Mapper转换到Reducer的过程
- 数据倾斜、数据处理慢 → 优化Shuffle

MapReduce Shuffle流程
- Map
    - map输出的数据会写入到缓冲区，并进行预排序的一些工作
    - 缓冲区：环形KvBuffer数据结构
    - 缓冲区还有Index数据，包含数据文件的位置、最后一行数据的位置、数据属于哪个分区等等
    - buffer会把数据写到硬盘上，多个map同时运行，buffer需要知道数据在哪个地方合并，因此需要Index
    - Index方便Map做合并等操作
- Spill
    - 缓冲区使用量达到一定比例后
        - 比例通过mapreduce.map.sort.spill.percent配置，默认0.8
    - 一个后台线程开始把缓冲区的数据写入磁盘，这个写入过程叫spill
    - map会继续将输出写入缓冲区
    - 如果缓冲区写满了，map会*阻塞*，直到apill过程完成，而不会覆盖缓冲区中已有的数据
    - spill过程是按照分区来组织数据的
        - Partition默认是按key的hash值来分的（预排序）
- 合并Spill文件
    - 减少reduce阶段的线程数，从而减少网络传输
    - map任务中，缓冲区达到设定的阈值，就会触发spill操作，因此会有多个spill文件
    - 在map任务结束之前，这些spill文件会根据情况合并到一个大的分区的、排序的文件中
    - 排序是在内存排序的基础上进行全局排序
    - 如果spill文件数量大于mapreduce.map.combiner.minspills配置的数（默认3），则在合并文件写入之前，会再次运行combiner
    - 如果spill文件数量太少，运行combiner的收益可能小于调用的代价
        - combiner：mapper端运行的reduce
    - mapreduce.task.io.sort.factor属性配置每次最多合并多少个文件，默认10
- 压缩
    - 数据量大的时候，对map输出进行压缩
    - 提升在shuffle阶段文件传输速度
    - 启用压缩，将mapreduce.map.output.compress设为true
    - 使用mapreduce.map.output.compress.codec设置使用的压缩算法
- Reduce
    - 复制map任务数据-copy过程
    - 维护几个copier线程，并行地从map任务机器提取数据，可以通过mapreduce.reduce.shuffle.parallelcopies配置（默认5）
    - 如果map输出的数据足够小，则会被拷贝到reduce任务的JVM内存
    - 配置JVM堆内存的多少比例可以用于存放map任务的输出结果：mapreduce.reduce.shuffle.input.buffer.percent（默认0.7）
- 内存中合并
    - 当缓冲中数据打到配置的阈值时，这些数据在内存中被合并、写入机器磁盘
    - 当内存满了，则直接写入磁盘
    - 配置阈值有2种方式，任意一个满足就会触发合并写入
        - 配置内存比例
            - reduce JVM堆内存的一部分用于存放来自map任务的输入
            - 假设用于存放map输出的内存为300M
            - mapreduce.reduce.shuffle.merger.percent配置为0.80
            - 则当内存中数据达到240M时，会触发合并写入
        - 配置map输出数量
            - mapreduce.reduce.merge.inmem.threshold
            - 如：大于1000M即触发合并写入
- 最终磁盘中合并

MapReduce Shuffle优化
- Map优化
    - 用于map输出排序的内存大小（默认100M）：mapreduce.task.io.sort.mb（根据数据块大小调整）
    - 开始spill的缓冲池阈值（默认0.8）：mapreduce.map.sort.spill.percent
    - 运行combiner的最低spill文件数量（默认3）：mapreduce.map.combine.minspills
    - 合并文件数最大值，与reduce公用（默认10）：mapreduce.task.io.sort.factor
    - 输出是否压缩（默认false）：mapreduce.map.out.compress
    - 压缩格式：mapreduce.map.output.compress.codec（Snappy, LZO等）
- Reduce优化
    - 提取map输出的copier线程数（默认5）：mapreduce.reduce.shuffle.parallelcopies（根据集群数以及CPU使用率调整，一般是两倍集群数）
    - 提取map输出最大尝试次数，超出后报错（默认10）：mapreduce.reduce.shuffle.maxfetchfailures
    - 合并文件数最大值，与map公用（默认10）：mapreduce.task.io.sort.factor
    - copy阶段用于保存map输出的堆内存比例（默认0.7）：mapreduce.reduce.shuffle.input.buffer.percent
    - 开始spill的缓冲池比例阈值（默认0.66）：mapreduce.reduce.shuffle.merge.percent
    - 开始spill的map输出文件数量阈值，小于等于0表示没有阈值，此时只由比例阈值来控制（默认1000）：mapreduce.reduce.shuffle.inmem.threshold

***

<h4 id='2'>第二节 使用MRUnit进行单元测试过程</h4>

1. 掌握MapReduce单元测试方法
2. 能够使用MapReduce单元测试完成项目测试

---

MRUnit
- Couldera公司开发，专门针对Hadoop中编写MapReduce单元测试的框架
- MapDriver
- ReduceDriver
- MapReduceDriver

***

<h4 id='3'>第三节 MapReduce 数据压缩Snappy,Gzip,LZO</h4>

1. 掌握MapReduce压缩原理
2. 掌握MapReduce压缩配置方法

---

MapReduce压缩
- 文件压缩好处
    - 节省文件存储空间
    - 加速网络数据传输或磁盘读写
- 常用压缩格式
    压缩格式|工具|算法|扩展名|多文件|可分割性
    -|-|-|-|-|-
    DEFLATE|无|DEFLATE|.deflate|不|不
    GZIP|gzip|DEFLATE|.gzp|不|不
    ZIP|zip|DEFLATE|.zip|是|是，在文件范围内
    BZIP2|bzip2|BZIP2|.bz2|不|是
    LZO|lzop|LZO|.lzo|不|是
- 压缩效果对比
    压缩算法|原始文件大小|压缩文件大小|压缩速度|解压速度
    -|-|-|-|-
    gzip|8.3GB|1.8GB|17.5MB/s|58MB/s
    bzip2|8.3GB|1.1GB|2.4MB/s|9.5MB/s
    LZO-bset|8.3GB|2GB|4MB/s|60.6MB/s
    LZO|8.3GB|2.9GB|49.3MB/s|74.6MB/s
- 压缩算法说明
    压缩格式|split|native|压缩率|速度|是否hadoop自带|linux命令|换成压缩格式后，原来的应用程序是否要修改
    -|-|-|-|-|-|-|-
    gzip|否|是|很高|比较快|是|有|和文本处理一样，不需要修改
    lzo|是|是|比较高|很快|否|有|需要建索引，还需要指定输入格式
    snappy|否|是|比较高|很快|否|没有|和文本处理一样，不需要修改
    bzip2|是|否|最高|慢|是|有|和文本处理一样，不需要修改
- bzip2压缩比最高，压缩效率低，可以用作数据归档
- lzo压缩比低，压缩效率高，并且可分割，适合生产环境
    - CDH默认实现LZO，其他版本Hadoop需要安装LZO
    - 生产环境常用ORCFile+LZO
    - 优点
        - 压缩/解压速度比较快，合理的压缩率
        - 支持split，是Hadoop中最流行的压缩格式
        - 支持Hadoop native库
        - 可以在linux系统下安装lzop命令，使用方便
    - 缺点
        - 压缩率比gzip要低一些
        - Hadoop本身不支持，需要安装
        - 应用中，对lzo格式的文件要做一些特殊处理
            - 支持split需要建索引
            - 需要指定inputformat为lzo格式
- 在不需要分割的情况下（shuffle过程一般都需要将数据分割成split），生产环境可选择gzip或snappy
    - 数据压缩后文件大于1G，选择LZO
    - 数据压缩后文件大于1G，选择gzip或snappy

MapReduce压缩使用
- Map端压缩
    - mapreduce.map.output.compress
    - mapreduce.map.output.compress.codec
- MapReduce端输出压缩
    - mapreduce.output.fileoutputformat.compress
    - mapreduce.output.fileoutputformat.compress.type=BLOCK
        - 默认情况下是RECORD，即针对每条记录进行压缩
        - 如果改成BLOCK，将针对一组记录进行压缩
        - 数据量大，采用BLOCK，压缩效率高，压缩比低
    - mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.GzipCodec

***

<h4 id='4'>第四节 MapReduce Partitioner,Combiner实现及应用</h4>

1. 掌握MapReduce Partitioner
2. 掌握MapReduce Combiner

---

MapReduce Partitioner
- Hadoop中的Partitioner阶段，针对Map阶段输出的数据进行分区处理
- 对key进行分区处理，使拥有不同的key的数据被分到不同的Reduce中处理
- 其处理位置是在节点本身
- 可以在Reduce之前进行一次分类，提高效率
- 可以自定义key的分区规则，如数据文件包含不同省份，每个省份输出一个文件
- 框架本身有默认的HashPartitioner

MapReduce Combiner
- 在map端对输出先做一次合并
- 实现本地key的归并，类似本地reduce功能
- job.setCombinerClass(MyReducer.class)

***

<h4 id='5'>第五节 Mapreduce高级编程</h4>

1. 掌握MapReduce数据去重
2. 掌握MapReduce数据排序
3. 掌握MapReduce倒排索引

---

数据去重
- 利用MapReduce对key的汇聚机制，将重复的数据去掉

二次排序
- Hadoop默认将结果mapper的输出按照key来进行排序
- 二次排序：将数据按照value排序
    - 构造一个WritableComparable对象，同时保留Key和Value的值
    - 自定义方法中的排序方法compareTo

倒排索引
- 统计每个关键词在各个文档当中的第几行出现了几次
