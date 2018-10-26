# 目录 #

- [第一节 搭建Kafka开发环境](#1)
- [第二节 开发Kafka的消息发送和接收组件代码](#2)
- [第三节 Kafka的数据持久化](#3)
- [第四节 Kafka与Spark Streaming进行整合开发](#4)
- [第五节 Kafka高可用实现原理数据高可用保证](#5)
- [第六节 Kafka性能优化](#6)

***

<h4 id='1'>第一节 搭建Kafka开发环境</h4>

1. 了解Kafka基本开发过程
2. 掌握Kafka开发环境搭建

---

开发环境搭建
- 导入相关类库即可
    ```
    <name>kafka.api</name>
    <url>http://maven.apache.org</url>
    ```
- 支持Java API、Scala API
- 不依赖Hadoop环境，可单独运行

***

<h4 id='2'>第二节 开发Kafka的消息发送和接收组件代码</h4>

1. 掌握Kafka消息发送者开发方法与API
2. 掌握Kafka消息消费者API

---

Kafka应用开发过程
- 发送者代码
    - 开发语言：Java、Scala
    - 相关类：kafka.javaapi.producer.Producer
```
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;

public class KafkaProducerAPI {

    // 1. 初始化Kafka服务
    // 服务器上启动Kafka，创建topic
    // 2. Properties参数配置等
    // 3. KafkaProducerAPI发送消息

    private final String TOPIC = "testAPI";
    private Properties props = new Properties();

    public  KafkaProducerAPI(){}

    public void run() {
        // set key serializer
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        // set value serializer
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        // set codec compression
        props.put("compression.codec", "gzip");
        // set brokers
        props.put("bootstrap.servers", "hadoop001:9092,hadoop:9093");

        // init Kafka Producer
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(props);

        int data = 1;
        while (data < 200) {
            String msg = new String(data + "_msg");
            try {
                Thread.sleep(1000); // 防止发送太快导致漏数据
                producer.send(new ProducerRecord<String, String>(TOPIC, Integer.toString(data), msg));
                System.out.println(msg);
                data++;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        // close Producer
        producer.close();
    }

    public static void main(String[] args) {
        new KafkaProducerAPI().run();
    }

}
```
- 消费者代码
    - 开发语言：Java、Scala
    - 相关类：kafka.javaapi.comsumer.ConsumerConnector
    - 相关类：kafka.consumer.ConsumerIterator
    - 相关类：kafka.consumer.ConsumerConfig
```
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.util.Arrays;
import java.util.Properties;

public class KafkaConsumerAPI {
    // 1. setup topic
    private final String TOPIC = "testAPI";
    private final Properties props = new Properties();

    public KafkaConsumerAPI(){}

    public void run() {
        // 2. properties
        props.put("bootstrap.servers", "hadoop001:9092,hadoop:9093");
        props.put("zookeeper.connect", "hadoop001:2181");
        // set up group
        props.put("group.id", "test");
        // set deserializer
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        props.put("enable.auto.commit", "true");
        props.put("auto.commit.interval.ms", "1000");

        // 3. KafkaConsumer
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
        consumer.subscribe(Arrays.asList(TOPIC));
        while(true) {
            ConsumerRecords<String, String> records = consumer.poll(1000);
            for (ConsumerRecord<String, String> record : records) {
                System.out.println(record.offset() + ": " + record.value());
            }
        }
    }

    public static void main(String[] args) {
        new KafkaConsumerAPI().run();
    }

}
```
- 多线程Consumer
```
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.util.Arrays;
import java.util.Properties;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class KafkaMutiConsumerDemo {

    private final String TOPIC = "testAPI";
    private KafkaConsumer<String, String> consumer = null;
    private ExecutorService executorService;

    public KafkaMutiConsumerDemo() {
        Properties props = new Properties();
        props.put("bootstrap.servers", "hadoop001:9092,hadoop:9093");
        props.put("zookeeper.connect", "hadoop001:2181");
        // set up group
        props.put("group.id", "group");
        // set deserializer
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        props.put("enable.auto.commit", "false"); // 不自动提交
        props.put("auto.commit.interval.ms", "1000");

        consumer = new KafkaConsumer<String, String>(props);
        consumer.subscribe(Arrays.asList(TOPIC));
    }

    public void execute() {
        executorService = Executors.newFixedThreadPool(3); // 3线程
        while(true) {
            ConsumerRecords<String, String> records = consumer.poll(1000);
            if (null != records) {
                executorService.submit(new ConsumerThread(records, consumer));
            }
        }
    }

    public void shutdown() {
        if(consumer != null)
            consumer.close();
        if(executorService != null)
            executorService.shutdown();
    }

    public static void main(String[] args) {
        new KafkaMutiConsumerDemo().execute();
    }
}
```
```
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.TopicPartition;

import java.util.List;

public class ConsumerThread implements Runnable {

    private ConsumerRecords<String, String> records;
    private KafkaConsumer<String, String> consumer;
    public ConsumerThread(ConsumerRecords<String, String> records, KafkaConsumer<String, String> consumer) {
        this.records = records;
        this.consumer = consumer;
    }

    public void run() {
        // thread = partition: 每个线程读一个分区
        for(TopicPartition partition :records.partitions()) {
            List<ConsumerRecord<String, String>> partitionsRecords = records.records(partition);
            for(ConsumerRecord<String, String> record: partitionsRecords) {
                System.out.println(record.offset() + ": " + record.value());
            }
        }
    }
}
```

***

<h4 id='3'>第三节 Kafka的数据持久化</h4>

1. 掌握Kafka数据持久化方法
2. 掌握Kafka持久化原理及性能提升的方法

---

数据持久化过程
- 将数据从内存写入磁盘（数据库）

Kafka数据持久化高性能原理
- 高效实用磁盘
    - 顺序写磁盘：性能高于随机写内存
    - Append Only：数据不更新
    - 无记录级的数据删除（只会整个segment删除）
    - 充分利用Page Cache
        - 页高速缓冲存储器（页高缓pcache）
        - page cache的大小为一页，通常为4K
        - 在linux读写文件时，它用于缓存文件的逻辑内容，从而加快对磁盘上映像和数据的访问
    - 支持多个Directory（可使用多Drive）
- 零拷贝
- 批处理和压缩
    - Producer和Consumer均支持批量处理数据，减少网络传输的开销
    - Producer可将数据压缩后发送给broker，从而减少网络传输的开销
        - 目前支持Snappy、Gzip和LZ4压缩
- Partition
    - Partition实现了并行处理和水平扩展
    - Partition是Kafka（包括Kafka Stream）并行处理的最小单位
    - 不同Partition可处于不同的Broker（节点），充分利用多机资源
    - 同一Broker（节点）上的不同Partition可置于不同的Directory，如果节点上有多个Disk Drive，可将不同的Drive对应不同的Directory，从而使Kafka充分利用多Disk Drive的磁盘优势
- ISR(In-Sync Replicas)
    - 通过副本机制提供高可用
    - 通过多副本（副本Leader）提高读的速度

***

<h4 id='4'>第四节 Kafka与Spark Streaming进行整合开发</h4>

1. 掌握Kafka与Spark Streaming整合开发
2. 掌握Kafka与Spark Checkpoint机制
3. 掌握如何进行偏移量保存

---

- spark自己维护offset，消除了offset维护不一致的问题
```
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.spark.SparkConf;
import org.apache.spark.streaming.Durations;
import org.apache.spark.streaming.api.java.JavaInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka010.HasOffsetRanges;
import org.apache.spark.streaming.kafka010.KafkaUtils;
import org.apache.spark.streaming.kafka010.OffsetRange;
import org.apache.spark.streaming.kafka010.CanCommitOffsets;
import org.apache.spark.streaming.kafka010.ConsumerStrategies;
import org.apache.spark.streaming.kafka010.LocationStrategies;

import java.util.Arrays;
import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

public class KafkaSpark {

    public static void main(String[] args) {
        SparkConf conf = new SparkConf().setAppName("test").setMaster("local[3]");
        JavaStreamingContext jssc = new JavaStreamingContext(conf, Durations.seconds(2));
        // create topic
        String kafkaTopics = "testAPI";
        Collection<String> topics = Arrays.asList(kafkaTopics);
        // setup kafka params
        Map<String, Object> kafkaParams = new HashMap<String, Object>();
        kafkaParams.put("bootstrap.servers", "hadoop001:9093");
        kafkaParams.put("key.deserializer", StringDeserializer.class);
        kafkaParams.put("value.deserializer", StringDeserializer.class);
        kafkaParams.put("group.id", "test");
        kafkaParams.put("auto.offset.reset", "latest");
        // latest: ssc重新启动时，读最新的offset，会丢失部分数据
        // earliest: ssc重新启动时，从提交的offset开始消费，没有提交时从头开始，会重复消费数据
        // none: ssc重新启动时，topic各分区都存在已提交的offset时，从offset开始消费，只要有一个分区不存在提交，则抛出异常
        kafkaParams.put("enable.auto.commit", false); // 消费完不自动提交
        // kafka读一批数据，塞入数据库，如果塞入失败，重新读取这批数据
        // true: 读的时候已经更新了offset，不容易重新读
        // false: 只有更新成功才提交

        JavaInputDStream<ConsumerRecord<String, String>> stream = KafkaUtils.createDirectStream(
                jssc,
                LocationStrategies.PreferConsistent(),
                ConsumerStrategies.<String, String>Subscribe(topics, kafkaParams)
        );

        stream.foreachRDD(rdd->{
            rdd.foreach(x->{
                System.out.println(x);
            });
            OffsetRange[] offsetRanges = ((HasOffsetRanges) rdd.rdd()).offsetRanges();
            ((CanCommitOffsets) stream.inputDStream()).commitAsync(offsetRanges);
        });

    }

}
```

调优
- 合理的批处理时间（batchDuration）
    - SSC读取数据的间隔要和Kafka处理的速度相匹配
    - 需要进行集群测试（一般是10s）
- 合理的Kafka拉取量
- 缓存反复使用的DStream（RDD）
- 合理的GC（使用CMS，-XX:+UseConcMarkSweepGC）
- 合理的CPU资源数
- 合理的parallelism

checkpoint
```
jssc.checkpoint("");
```

***

<h4 id='5'>第五节 Kafka高可用实现原理数据高可用保证</h4>

1. 掌握Kafka HA实现原理数据高可用保证方法
2. 掌握Kafka HA搭建使用

---

Replication：数据高可用
- 某个Topic的replication-factor为N且N大于1时，每个Partition都会有N个副本（Replica）
- Replica的个数小于等于Broker数，即对每个Partition而言每个Broker上只会有一个Replica，因此可用BrokerID表示Replica
- 所有Partition的所有Replica默认情况会均匀分布到所有Broker上
- 副本有一个Leader，N个Follower，只针对Leader进行写操作，保证数据的一致性
- Leader选举
    - Leader写成功就返回，Leader等待Follower写成功，再返回
    - ISR：In-sync replication
    - Leader维护一个副本队列，包括自己，会将响应慢的Follower删除，追上的Follower再加回来
        - Follower从Leader收入数据，本质是异步的
        - replica.lag.max.messages=4000 Follower落后Leader4000条数据时，判定为较慢，将Follower从内存中移除
        - Kafka不保证数据不重复：写入Leader，写完了正好断网，但是Follower已经在同步数据
            - ack:request.required.acks=-1
            - producer.type=sync

Broker高可用
- Controller负责Partition Leader的选举，增删Topic时，replica重新分配
- Broker启动时，会在Zookeeper注册一个节点，同时还有一个watch（Zookeeper触发器）
- 一旦宕机，剩下的节点会注册新的Broker，然后会在Zookeeper的state目录下重新选择partition Leader

***

<h4 id='6'>第六节 Kafka性能优化</h4>

1. 掌握Kafka常见性能优化方法
2. 掌握kafka性能优化参数

---

Partition数量配置
- Partition数量有Topic的并发决定，并发少则1个分区，并发越高，分区数越多，可以提高吞吐量
- 越多的partition，需要IO更高
- 一般需要提前规划partition数量，先创建一些空的
- 每一个broker：partition2000-4000个
- 每一个Kafka集群：限制在10000万以内
- 以增加Broker的方式增加partition，而不是在某一个Broker里增加

日志保留策略设置
- 默认保留7天，建议根据磁盘情况配置，避免磁盘撑爆
- 段文件配置1GB，有利于快速回收磁盘空间，重启Kafka加载也会加快

文件刷盘策略
- 为了大幅度提高producer写入吞吐量，需要定期批量写文件段文件

设置压缩
- compression.type：针对全局
- compression.codec：针对当前producer
- compression.topics：针对目前的