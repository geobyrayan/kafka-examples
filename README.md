# kafka-examples

These examples written in __Java__ demostrate different concepts of __Kafka__ messaging, in a __simple__ manner.

Motivation for these examples comes from [Ryan Plant's](https://twitter.com/ryan_plant) Kafka [course](https://app.pluralsight.com/library/courses/apache-kafka-getting-started/table-of-contents). 

Example code expects a minimal setup, for which sequential instructions are mentioned in __Setup__ section below. Please follow these instructions before running the code.

Examples are arranged by following topics:

+ Warmup with console utilities
+ [Basic Producer and Baisc Consumer](https://github.com/agrawalnishant/kafka-examples/blob/master/README.md#setup)
+ [Consumer Group](https://github.com/agrawalnishant/kafka-examples#multi-partition-setup-for-consumer-group)
+ Custom Partitioner
+ [Schema Registery](https://github.com/agrawalnishant/kafka-examples#setup-for-schema-registry)

## Setup
### Basic Single-Partition Setup
1. To use some advanced features (i.e. Schema Registry) of Kafka, we will use Confluent's distribution available in [Download Center](https://www.confluent.io/download-center/). Download and un-Tar the contents. Lets call this location as KAFKA_HOME, for the purpose of discussion here. This location has folders like bin/ and config/ inside.

2. In console / terminal window go to KAFKA_HOME location, and execute following commands:
    1. Start zookeeper:
    `zookeeper-server-start.sh ../config/zookeeper.properties`
        - Wait for Zookeeper to start.
    2. Start Kafka broker/server:
        `kafka-server-start.sh ../config/server.properties`
        
3. Create a Kafka topic `topic_basic` with single replica and single partition:
    - `kafka-topics.sh  -zookeeper localhost:2181 --create --topic topic_basic --replication-factor 1 --partitions 1`
    
[Source Code](https://github.com/agrawalnishant/kafka-examples/tree/master/src/main/java/kafka/examples/basic)
        
### Multi-partition Setup for Consumer Group
Step# 2 above would start a single Zookeeper instance and a single Kafka broker, which is enough for barebones message production and consumption. 

But for a more robust and fault-tolerant Kafka Setup, we need at least 3 replicas and 3 partitions of Kafka for single failover instance. [ [Reason](https://forums.couchbase.com/t/why-3-node-cluster-for-automatic-failover/2759)]

__Create 2 partitions__

Single partition can only attach to a single Kafka consumer in a group setting. So, lets start one more broker to support a Consumer Group with 2 consumers. 
* Each new broker instance needs its own `server.properties` file. 
  So clone the existing config/server.properties to, say `server-1.properties`, and change these parameters:
    - `log.dirs` to new log directoy (say /tmp/kafka-logs-1)
    - `listeners` to new port (say, 9093)
    Keep zookeeper connect port same as earlier.
 * Use following command, from bin folder, to start a Kafka replica synced to same zookeeper:
     - `kafka-server-start.sh ../config/server-1.properties`
        
__Enable 2 Partitions for Topic to support Consumer Group__
* Use following command to increase replica count of topic, created ealier, to 2:
    - `kafka-topics.sh  -zookeeper localhost:2181 --alter --topic topic_basic --partitions 2`
    
* In the output log, we see there are 2 different thread ids and 2 different consumer ids, similar to these:

  `[pool-1-thread-1] [consumer:0d86c38a-7936-4df8-9a88-92d947d6e087]`
  
  `[pool-1-thread-2] [consumer:e53155d8-c59b-4820-81e2-846e70cb2b9a]`



[Source Code](https://github.com/agrawalnishant/kafka-examples/blob/master/src/main/java/kafka/examples/basic/StringProducerConsumerGroupDemo.java)    

### Setup for Schema Registry

In any messaging system, it is important to keep producers and consumers to agree on message schema. If a message enters messaging system that is not valid for a consuming application, it will add overhead on consumers to handle such invalid messages.

This point is stressed enough in [Gwen Shapira's tech talk](https://vimeo.com/167028700). [Confluent](https://www.confluent.io/) provides a clean standardized solution for this in Schema Registry, which is shown by [Schema Registry example code](https://github.com/agrawalnishant/kafka-examples/tree/master/src/main/java/kafka/examples/schema/registry).

Most messaging and streaming systems use Avro, for reasons shared by [Cloudera](http://blog.cloudera.com/blog/2011/05/three-reasons-why-apache-avro-data-serialization-is-a-good-choice-for-openrtb/), and on [Quora](https://www.quora.com/What-are-pros-and-cons-of-Apache-Avro).

Please follow these steps for SchemaRegisteryDemo to work:
* We use a different Topic for messages based on a particular schema:

Go to bin folder to create a new Kafka topic `topic_schema` with 2 partitions, while 2 kafka broker instances are running:
  - `kafka-topics.sh -zookeeper localhost:2181 --create --topic topic_schema --replication-factor 1 --partitions 2`


* Start Schema Registry
  
  Go to $KAFKA_HOME/bin, and execute:
    - `schema-registry-start ../etc/schema-registry/schema-registry.properties`
    
    This is the reason we need the Confluent's distribution of Kafka. 
    
* Compile Avro Schema (KafkaExampleMessage.avsc) to Java class:
    - `mvn generate-sources`


[Source Code](https://github.com/agrawalnishant/kafka-examples/tree/master/src/main/java/kafka/examples/schema/registry)
