

- KafkaConsumer is not thread-safe, KafkaProducer is thread safe.
- In Kafka, a small heap size is needed, while the rest of the RAM goes automatically to the page cache (managed by the OS). The heap size goes slightly up if you need to enable SSL
- JDBC connector allows one task per table.
- Partition leader election is done by - controller
- Kafka partitions are made of segments (usually each segment is 1GB), and each segment has two corresponding indexes (offset index and time index)
- If you enable an SSL endpoint in Kafka -> zero copy is lost -> check further
- We can only add partitions to an existing topic, and it must be done using the kafka-topics.sh command
- Default port of KSQL server is 8088

- Dynamic topic and broker configs are stored in zookeeper and doesn't need restart
- acks is a producer setting min.insync.replicas is a topic or broker setting and is only effective when acks=all

- In case the consumer or producer has the wrong leader of a partition, it will issue a metadata request. The Metadata request can be handled by any node, so clients know afterwards which broker are the designated leader for the topic partitions. Produce and consume requests can only be sent to the node hosting partition leader.

- ACLs are stored in zookeeper under `/kafka-acl/` node by default

#### Retriable ERRORS

- https://kafka.apache.org/protocol#protocol_error_codes



#### Default ports

- Zookeeper Client Port: 2181
- Zookeeper Leader Port: 3888
- Zookeeper Election Port (Peer port): 2888
- Broker: 9092
- REST Proxy: 8082
- Schema Registry: 8081
- KSQL: 8088


#### Kakfa meta topics
- __consumer_offsets
- _schemas


- Tests
3:22 ,
