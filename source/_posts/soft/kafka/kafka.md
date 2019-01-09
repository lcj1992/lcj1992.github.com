---
layout: post
title: kafka源码分析
date: 2016-07-10
categories: soft
tags: kafka
---




### kafka源码安装

    下载源码，而不是binaryhttp://kafka.apache.org/downloads.html
    gradle    // 编译
    ./gradlew idea    // 导入到idea

### 基本概念

1. producer 生产者
2. consumer 消费者
3. topic  主题
4. broker 代理、中间人
5. partition 分区
6. segment 段
7. consumer group 消费组

### 使用

首先参照[官网quickstart](http://kafka.apache.org/documentation/#quickstart)找到代码对应的入口。

1. 启动server。
   * 主类为`kafka.Kafka`
   * 启动kafka前需要先起zk。
   * 从github上拉下来的代码，没有log4j的配置，需要拷贝一份，
   * 启动参数`/path/to/server.properties`,指定配置文件位置
2. 创建topic
   * 主类为`kafka.admin.TopicCommand`
   * 启动参数`--create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test`
3. producer发送消息
   * 以concole producer为例，主类为`kafka.tools.ConsoleProducer`
   * 启动参数`--broker-list localhost:9092 --topic test`
4. consumer接受消息
   * 以console consumer为例，主类为`kafka.tools.ConsoleConsumer`
   * 启动参数`--bootstrap-server localhost:9092 --topic test --from-beginning`

### 文件存储

### 参考

[kafka官方文档](http://kafka.apache.org/documentation)

[kafka背景及架构介绍](http://www.infoq.com/cn/articles/kafka-analysis-part-1)

[Kafka High Availability(上)](http://www.infoq.com/cn/articles/kafka-analysis-part-2)

[Kafka High Availability(下)](http://www.infoq.com/cn/articles/kafka-analysis-part-3)

[kafka consumer解析](http://www.infoq.com/cn/articles/kafka-analysis-part-4)

[kafka benchmark](http://www.infoq.com/cn/articles/kafka-analysis-part-5)

[kafka文件存储那些事](http://tech.meituan.com/kafka-fs-design-theory.html)
