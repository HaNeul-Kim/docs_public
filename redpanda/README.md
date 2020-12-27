# Redpanda

* https://vectorized.io/
* kafka api 와 호환됨
* C++ 로 쓰여진 kafka 라고 보면 됨
* zookeeper 대신 raft algorithm 을 사용함

VM 을 아래 형태로 준비함

|Hostname|IP|Role|
|:---:|:---:|:---|
|rpm.sky.local|192.168.181.170|Redpanda root node|
|rp1.sky.local|192.168.181.171|Redpanda worker node1|
|rp2.sky.local|192.168.181.172|Redpanda worker node2|
|rp3.sky.local|192.168.181.173|Redpanda worker node3|

## VM Preparation

```shell script
java -jar E:\vm\CopyVMWare-0.1-SNAPSHOT.jar --sourcePath E:\vm\linux --sourceVMName template76 --targetPath F:\vm\linux\redpanda --targetVMNames rpm,rp1,rp2,rp3
```

```shell
echo '
# Redpanda
192.168.181.170    rpm.sky.local    rpm
192.168.181.171    rp1.sky.local    rp1
192.168.181.172    rp2.sky.local    rp2
192.168.181.173    rp3.sky.local    rp3
' >> /etc/hosts
for h in w1 w2 w3 ; do scp /etc/hosts rp${h}.sky.local:/etc ; done
```

## Trial license

[download-trial](https://vectorized.io/download-trial/)

### Sign up for evaluation 버튼 클릭

### Company Name 입력

DataDynamics

### Work email 입력

haneul.kim@data-dynamics.io

### Do you accept out terms of use?

Accept 체크

확인

메일로 30일짜리 license key 가 옴

## Multi Node Production Setup

* https://vectorized.io/docs

### Step 1: Install the binary

모든 node 에서 실행함

```shell
mkdir ~/Downloads
cd ~/Downloads
curl -s https://4ea388d7df71a589f4325f7f344f9a65b6b70f56c51c3acf:@packagecloud.io/install/repositories/vectorizedio/v/script.rpm.sh | bash
yum install -y redpanda
rpk config set license_key eyJvIjoiRGF0YUR5bmFtaWNzIiwieSI6MjAyMCwibSI6MTIsImQiOjIwLCJjIjoyOTA2Mzk3NjM2fQ==
```

### Step 2: Start the root node

--self: ip. hostname 불가

```shell
rpk config bootstrap --id 0 --self 192.168.181.170
systemctl start redpanda-tuner redpanda
```

### Step 3: Start the other nodes

--self: ip. hostname 불가

```shell
for i in {1..3} ; do ssh rp${i} "rpk config bootstrap --id ${i} --self 192.168.181.17${i} --ips 192.168.181.170" ; done
for i in {1..3} ; do ssh rp${i} "systemctl start redpanda-tuner redpanda" ; done
```

## topic 생성

* https://vectorized.io/docs/rpk-commands/#api

```shell
rpk api topic list
rpk api status
rpk api topic create test_topic --compact -p 3 -r 1
rpk api topic describe test_topic --watermarks
rpk api topic list
rpk api status
```

```text
[root@rpm:~]# rpk api topic list
No topics found.
[root@rpm:~]# rpk api topic create test_topic --compact -p 3 -r 1
Created topic 'test_topic'. Partitions: 3, replicas: 1, configuration:
'cleanup.policy':'compact'
[root@rpm:~]# rpk api topic list
  Name        Partitions  Replicas
  test_topic  3           1
[root@rpm:~]# rpk api topic describe test_topic --watermarks
  Name                test_topic
  Internal            false
  Config:
  Name                Value       Read-only  Sensitive
  partition_count     3           false      false
  replication_factor  1           false      false
  Partitions          1 - 3 out of 3
  Partition           Leader          Replicas   In-Sync Replicas  High Watermark
  0                   1               [1]        [1]               1
  1                   2               [2]        [2]               1
  2                   3               [3]        [3]               1
[root@rpm:~]# 
```

## kafka api 를 활용한 데이터 생성

pom.xml

```xml
<!-- https://mvnrepository.com/artifact/org.apache.kafka/kafka -->
<dependency>
  <groupId>org.apache.kafka</groupId>
  <artifactId>kafka_2.13</artifactId>
  <version>${kafka.version}</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients -->
<dependency>
  <groupId>org.apache.kafka</groupId>
  <artifactId>kafka-clients</artifactId>
  <version>2.6.0</version>
</dependency>
```

```scala
package com.tistory.hskimsky.redpanda

import org.apache.kafka.clients.consumer._
import org.apache.kafka.clients.producer._
import org.apache.kafka.common.serialization.{StringDeserializer, StringSerializer}
import org.apache.kafka.common.{PartitionInfo, TopicPartition}
import org.junit.Test
import org.slf4j.LoggerFactory

import java.time.Duration
import java.util.{Base64, Properties}
import java.{lang, util}
import scala.collection.mutable
import scala.jdk.CollectionConverters._

class ProduceConsumeTest {

  private val logger = LoggerFactory.getLogger(classOf[ProduceConsumeTest])

  private val SERVERS = "rpm.sky.local:9092,rp1.sky.local:9092,rp2.sky.local:9092,rp3.sky.local:9092"

  @Test
  def redpandaLicenseTest(): Unit = {
    val licenseKey = "eyJvIjoiRGF0YUR5bmFtaWNzIiwieSI6MjAyMCwibSI6MTIsImQiOjIwLCJjIjoyOTA2Mzk3NjM2fQ=="
    val decoder: Base64.Decoder = Base64.getDecoder
    val decoded = new String(decoder.decode(licenseKey))
    logger.info(s"decoded = ${decoded}")
  }

  @Test
  def produceTest(): Unit = {
    val props: Properties = new Properties()
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, SERVERS)
    props.put(ProducerConfig.CLIENT_ID_CONFIG, "client1")
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, classOf[StringSerializer].getName)
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, classOf[StringSerializer].getName)
    val producer = new KafkaProducer[String, String](props)

    for (i <- 0 until 10000) {
      val key = s"k${i}"
      val value = s"v${i}"
      val record = new ProducerRecord[String, String]("test_topic", key, value)
      producer.send(record)
      //logger.info(s"key = ${key}, value = ${value}")
      //val recordMetadata: RecordMetadata = producer.send(record).get()
      //val recordMetadata: RecordMetadata = producer.send(record).get()
      //val partition: Int = recordMetadata.partition()
      //val offset: Long = recordMetadata.offset()
      //logger.info(s"key = ${key}, value = ${value}, partition = ${partition}, offset = ${offset}")

      producer.send(record, (metadata: RecordMetadata, exception: Exception) => {
        val topic: String = metadata.topic()
        val timestamp: Long = metadata.timestamp()
        val partition: Int = metadata.partition()
        val offset: Long = metadata.offset()
        logger.info(s"topic=${topic}, key=${key}, val=${value}, ts = ${timestamp}, partition = ${partition}, offset = ${offset}, exception = ${exception}")
      })
    }
    //producer.commitTransaction()
    producer.flush()
    producer.close()
  }

  @Test
  def partitionTest(): Unit = {
    val props: Properties = new Properties()
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, SERVERS)
    props.put(ConsumerConfig.CLIENT_ID_CONFIG, "client1")
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "group1")
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, classOf[StringDeserializer].getName)
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, classOf[StringDeserializer].getName)
    props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest") //latest|earliest|none
    props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true")

    val consumer = new KafkaConsumer[String, String](props)
    val topic = "test_topic"
    //consumer.subscribe(Seq(topic).asJava)
    val partitionList: util.List[PartitionInfo] = consumer.partitionsFor(topic)
    val partitions: mutable.Buffer[PartitionInfo] = partitionList.asScala

    //val topicPartitions: util.Collection[TopicPartition] = partitions.map(partitionInfo => new TopicPartition(topic, partitionInfo.partition())).asJavaCollection
    val topicPartitions: util.Collection[TopicPartition] = this.topicPartitions(consumer, topic)
    val beginningOffsetMap: util.Map[TopicPartition, lang.Long] = consumer.beginningOffsets(topicPartitions)
    val beginningOffsets: mutable.Map[TopicPartition, lang.Long] = beginningOffsetMap.asScala
    beginningOffsets.foreach(beginningOffset => {
      logger.info(s"beginningOffset = ${beginningOffset}")
    })

    val endOffsetMap: util.Map[TopicPartition, lang.Long] = consumer.endOffsets(topicPartitions)
    val endOffsets: mutable.Map[TopicPartition, lang.Long] = endOffsetMap.asScala
    endOffsets.foreach(endOffset => {
      logger.info(s"endOffset = ${endOffset}")
    })
  }

  def topicPartitions(consumer: KafkaConsumer[_, _], topic: String): util.Collection[TopicPartition] = {
    consumer.partitionsFor(topic).asScala.
      map(partitionInfo => new TopicPartition(topic, partitionInfo.partition())).
      asJavaCollection
  }

  class Balancer(consumer: KafkaConsumer[_, _]) extends ConsumerRebalanceListener {
    override def onPartitionsRevoked(partitions: util.Collection[TopicPartition]): Unit = {
      partitions.asScala.foreach(topicPartition => {
        logger.info(s"topicPartition = ${topicPartition}")
      })
    }

    override def onPartitionsAssigned(partitions: util.Collection[TopicPartition]): Unit = {
      val topicPartitionOffsetAndMetadataMap: util.Map[TopicPartition, OffsetAndMetadata] = consumer.committed(partitions.asInstanceOf[util.Set[TopicPartition]])
      topicPartitionOffsetAndMetadataMap.asScala.foreach(topicPartitionOffsetAndMetadata => {
        val (topicPartition: TopicPartition, offsetAndMetadata: OffsetAndMetadata) = topicPartitionOffsetAndMetadata
        logger.info(s"topicPartition = ${topicPartition}, offsetAndMetadata = ${offsetAndMetadata}")
      })
    }
  }

  @Test
  def consumeTest(): Unit = {
    val props: Properties = new Properties()
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, SERVERS)
    props.put(ConsumerConfig.CLIENT_ID_CONFIG, "client1")
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "group1")
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, classOf[StringDeserializer].getName)
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, classOf[StringDeserializer].getName)
    //props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest") //latest|earliest|none, redpanda 에서 안되는 듯
    //props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true") // redpanda 에서 안되는 듯

    val consumer = new KafkaConsumer[String, String](props)
    val topic = "test_topic"
    consumer.subscribe(Seq(topic).asJava)
    //consumer.subscribe(Seq(topic).asJava, new Balancer(consumer))
    consumer.poll(Duration.ofSeconds(1))
    //val dummyRecords: ConsumerRecords[String, String] = consumer.poll(Duration.ofSeconds(1))
    //logger.info(s"dummyRecords = ${dummyRecords}, dummyRecords.count = ${dummyRecords.count()}")
    consumer.seekToBeginning(this.topicPartitions(consumer, topic))
    //consumer.seekToEnd(this.topicPartitions(consumer, topic))
    while (true) {
      val records: ConsumerRecords[String, String] = consumer.poll(Duration.ofSeconds(1))
      records.asScala.foreach(record => {
        val key: String = record.key()
        val value: String = record.value()
        val partition: Int = record.partition()
        val offset: Long = record.offset()
        val timestamp: Long = record.timestamp()
        logger.info(s"k=${key},v=${value},p=${partition},o=${offset},t=${timestamp}")
      })
    }
  }
}
```

## References

* https://vectorized.io/
* https://vectorized.io/download-trial/
* https://vectorized.io/docs
