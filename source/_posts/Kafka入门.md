title: Kafka入门
author: Jie
categories:
  - 数据采集
date: 2022-11-16 21:53:54
tags:
---
> 实验目的
> 1、熟悉kafka操作的常用命令
> 2、熟练使用python编写kafka程序
> 3、熟练完成kafka与MySQL的交互
> 4、熟悉消费者订阅分区和手动提交偏移量的API
<!-- more -->
### 启动服务
使用Kafka之前需要先启动一个ZooKeeper服务，直接使用Kafka中包含的脚本
```shell
root@vm:/usr/local/kafka/bin# ./zookeeper-server-start.sh config/zookeeper.properties
```
新开一个终端，启动Kafka服务
```shell
root@vm:/usr/local/kafka/bin# ./kafka-server-start.sh config/server.properties
```
### 1． Kafka与MySQL的组合使用

读取student表的数据内容，将其转为JSON格式，发送给Kafka

![](/images/pasted-19.png)

![](/images/pasted-20.png)

从Kafka中获取Json格式数据，打印出来

![](/images/pasted-21.png)

### 代码
```python
# producer
from kafka import KafkaProducer
import json
import pymysql

def mysql():
    # 连接数据库
    connect = pymysql.Connect(
        host='192.168.234.131',
        port=3306,
        user='root',  # 数据库用户名
        passwd='password',  # 密码
        db='school',
        charset='utf8'
    )
    # 获取游标对象
    cursor = connect.cursor()
    # 查询数据
    sql = "select * from student"
    # 执行SQL语句
    result = cursor.execute(sql)
    # fetchall查询所有结果
    data = cursor.fetchall()
    for msg in data:
        val = {}
        val['sno'] = msg[0]
        val['sname'] = msg[1]
        val['ssex'] = msg[2]
        val['sage'] = msg[3]
        producer.send('json_topic', val)  # 发送的topic为json
        # 提交事务
        connect.commit()
        print(val)
    connect.close()

if __name__ == '__main__':
    producer = KafkaProducer(bootstrap_servers='192.168.234.131:9092',
                             value_serializer=lambda v: json.dumps(v).encode('utf-8'), api_version=(0, 10))  # 连接kafka
    mysql()
    producer.close()

# consumer
import json
from kafka import KafkaConsumer
from kafka.structs import TopicPartition

consumer = KafkaConsumer('json_topic', bootstrap_servers=[
                         '192.168.234.131:9092'], group_id=None, auto_offset_reset='earliest', api_version=(0, 10))

for message in consumer:
    msg1 = str(message.value, encoding="utf-8")
    dict = json.loads(msg1)
    print("%s:%d:%d:%s" % (message.topic, message.partition, message.offset, dict))
```

### 2． Kafka消费者手动提交

生成data.json文件
```json
[
    {
        "name": "Tony",
        "age": "21",
        "hobbies": [
            "basketball",
            "tennis"
        ]
    },
    {
        "name": "Lisa",
        "age": "20",
        "hobbies": [
            "sing",
            "dance"
        ]
    }
]
```
编写生产者程序，将JSON文件数据发送给Kafka
```python
def data():
    with open('data.json', 'r', encoding='utf8')as fp:
        data = json.load(fp)  # 读取json文件
        for dict in data:
            print(dict)
            producer.send('json_topic', dict)
```
![](/images/pasted-23.png)
编写消费者程序，读取Kafka的JSON格式数据，并重置偏移量，从第5个偏移量消费
```python
# 重置偏移量，从第5个偏移量消费
consumer.seek(TopicPartition(topic='json_topic', partition=0), 5)
for message in consumer:
    msg1 = str(message.value, encoding="utf-8")
    dict = json.loads(msg1)
    print("%s:%d:%d:%s" % (message.topic, message.partition, message.offset, dict))
```
![](/images/pasted-24.png)

### 3. Kafka消费者订阅分区

启动Kafka，手动创建主体“assign_topic”,分区数量为2
```shell
root@vm:/usr/local/kafka/bin# ./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 2 --topic assign_topic
```
编写生产者程序，以通用唯一标识符UUID作为消息，发送给主体”assign_topic”
```python
import uuid
from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers='192.168.234.131:9092', api_version=(0, 10))  # 连接kafka
msg = str(uuid.uuid1()).encode('utf-8')  # 发送内容,必须是bytes类型
producer.send('assign_topic', msg)  # 发送的topic为test
producer.close()
```
![](/images/pasted-25.png)

编写消费者程序，订阅主题的分区0，只消费分区0数据

编写消费者程序，订阅主题的分区1，只消费分区1数据
```python
from kafka import KafkaConsumer
from kafka.structs import TopicPartition

def main(partition):
    consumer = KafkaConsumer(bootstrap_servers=[
                             '192.168.234.131:9092'], group_id=None, auto_offset_reset='earliest', api_version=(0, 10))
    consumer.assign([TopicPartition('assign_topic', partition)])
    print(consumer.assignment())  # 获取当前消费者订阅的主题
    for msg in consumer:
        recv = "topic名称:%s  分区位置:%d  偏移量:%d  key=%s value=%s" % (
            msg.topic, msg.partition, msg.offset, msg.key, msg.value)
        print(recv)

if __name__ == '__main__':
    # main(partition=0)
    main(partition=1)
```
![](/images/pasted-26.png)

### 实验总结
Kafka 是由 Linkedin 公司开发的，它是一个分布式的，支持多分区、多副本，基于 Zookeeper 的分布式消息流平台，它同时也是一款开源的基于发布订阅模式的消息引擎系统，是一种高吞吐、低延迟的分布式发布订阅消息系统。
