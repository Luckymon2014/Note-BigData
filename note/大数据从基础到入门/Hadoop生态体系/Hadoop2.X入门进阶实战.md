# 目录 #

[HadoopEnv与YarnEnv配置](#1)
[HDFS生产环境配置文件解析](#2)
[MapReduce生产环境配置文件解析](#3)
[Yarn生产环境配置文件解析](#4)
[如何在不同配置的集群中合理管理Hadoop配置](#5)

***

120台机器（NameNode2台，DataNode118台）
机器配置：96G内存，24块1TB磁盘（SAS）

***

<h4 id='1'>HadoopEnv与YarnEnv配置</h4>

1. 了解hadoop-env.sh生产环境配置方法
2. 掌握yarn-env.sh重要的配置参数

hadoop-env.sh
- hadoop的运行环境
- JAVA_HOME
- HADOOP_CLASSPATH
- YARN_HOME
- HBASE_HOME
    - 一般在/usr/lib目录下
    - conf文件一般在/etc目录下
- HADOOP_HEAPSIZE
- HADOOP_NAMENODE_OPTS
    - 一般是一半内存，即：-Xms45g -Xmn4g
- HADOOP_DATANODE_OPTS

yarn-evn.sh
- YARN的日志路径
- YARN_RESOURCEMANAGER_OPTS
    - -Xmx4g -Xmn1g
- YARN_NODEMANAGER_OPTS
    - -Xmx2g -Xmn512m
- HADOOP_JOB_HISTORYSERVER_OPTS
    - 存储应用运行日志

***

<h4 id='2'>HDFS生产环境配置文件解析</h4>

1. 了解HDFS生产环境配置方法
2. 掌握HDFS常见更新的参数

磁盘目录配置
- dfs.namenode.name.dir
- dfs.datanode.data.dir

磁盘配置策略
- dfs.datanode.failed.volumes.tolerated
    - 磁盘损坏容忍数量
    - 一般是总量的1/4，即6
- dfs.datanode.fsdataset.volume.choosing.policy
    - 磁盘选择策略
    - 默认是轮询
    - org.apache.hadoop.hdfs.server.datanode.fsdataset.AvailableSpaceVolumeChoosingPolicy：均衡存储
- dfs.datanode.available-space-volume-choosing-policy.balanced-space-threshole
    - 磁盘使用空间最大值减去最小值，小于该值时就选择轮询策略
    - 10737418240（10G）
- dfs.datanode.available-space-volume-choosing-policy.balanced-space-preference-fraction
    - 磁盘使用容忍度
    - 一般是0.75（最多使用75%磁盘）

磁盘预留空间
- dfs.datanode.du.reserved
    - 每块磁盘保留的空余空间，预留给非hdfs文件使用
    - 和之前磁盘配置策略配合使用（10G）

主机数量包含管理
- dfs.hosts.exclude
    - 配置节点下线
- dfs.hosts
    - 配置节点上线

NameNode处理线程数量
- dfs.namenode.handler.count
    - 默认值很小，集群大时修改
    - namenode压力过大时，减少线程数

Block信息处理
- dfs.client.file-block-storage-locatioins.timeout
    - 定位block超时时间
- dfs.datanode.hdfs-blocks-metadata.enabled
    - 默认true

负载均衡
- dfs.balance.bandwidthPersec

***

<h4 id='3'>MapReduce生产环境配置文件解析</h4>

1. 了解MapReduce生产环境配置方法
2. 掌握MapReduce重要的配置参数

任务执行参数
- mapreduce.map.java.opts
    - -Xmx2048m
- mapreduce.reduce.java.opts
    - -Xmx4096m
- mapreduce.reduce.memory.mb
    - 4096
    - 一次能申请4G内存

目录配置
- mapreduce.jobhistory.intermediate-done-dir
- mapreduce.jobhistory.done-dir
    - jobhistory日志，任务执行情况
    - 一般配置在/user/history目录下

framework与端口配置
- mapreduce.framework.name
    - yarn
- mapreduce.jobhistory,address
    - db-cdh-218:10020
- mapreduce.jobhistory.webapp.address
    - db-cdh-218:19888

***

<h4 id='4'>Yarn生产环境配置文件解析</h4>

1. 了解Yarn生产环境配置方法
2. 掌握Yarn重要的配置参数

应用Classpath配置
- yarn.application.classpath
    - 将yarn用到的配置文件和lib配置进来

log配置
- yarn.nodemanager.local-dirs
    - 中间结果
- yarn.nodemanager.log-dirs
    - 日志，放到每个磁盘上，定期清理
- yarn.ndoemanager.remote-app-log-dir
- yarn.app.mapreduce.am.staging-dir

内存和CPU配置
- yarn.nodemanager.resource.memory-mb
    - 能够使用多少内存
    - 一般是1/2，即40960（40G）
- yarn.nodemanager.resource.cpu-vcores
    - 18

App计算资源配置
- yarn.scheduler.minimum-allocation-mb
    - 最小申请资源
    - 一般2048（2G）
- yarn.nodemanager.pmem-check-enabled
- yarn.nodemanager.vmem-check-enabled
    - 是否检查
    - 一般false

各端口分开配置
- 一般配置到90xx

***

<h4 id='5'>如何在不同配置的集群中合理管理Hadoop配置</h4>

1. 了解如何在不同配置的集群中合理管理Hadoop配置
2. 掌握为什么生产环境中会出现不同的配置

为什么会出现不同的配置
- 作用不同：离线分析（存储要求高，数量多）/在线分析（性能要求高，内存大）
- 采购时间不同，不同时间搭建

如何管理不同配置的集群——运维工具
- Puppet
    - 管理配置文件
    - 是一个开源的软件自动化配置和部署工具
    - 为C/S架构
    - 服务器端保存着所有对客户端服务器的配置代码
    - 与SVN一起使用
    ```mermaid
    graph TD
    user-->|用户提交作业|SVN(SVN DB)
    SVN-->|从SVN读配置|PM(Puppet Master)
    PM-->|推送配置|PC1(Puppet Client)
    PM-->|推送配置|PC2(Puppet Client)
    PM-->|推送配置|PC3(Puppet Client)
    ```
- SaltStack
    - 部署和配置工具
- Ansible
    - 服务的启动和管理