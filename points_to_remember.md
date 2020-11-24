

- KafkaConsumer is not thread-safe, KafkaProducer is thread safe.
- In Kafka, a small heap size is needed, while the rest of the RAM goes automatically to the page cache (managed by the OS). The heap size goes slightly up if you need to enable SSL
- JDBC connector allows one task per table.
- Partition leader election is done by - controller
- Kafka partitions are made of segments (usually each segment is 1GB), and each segment has two corresponding indexes (offset index and time index)
- If you enable an SSL endpoint in Kafka -> zero copy is lost -> check further
- We can only add partitions to an existing topic, and it must be done using the kafka-topics.sh command
- Default port of KSQL server is 8088


#### Default ports

- Zookeeper Client Port: 2181
- Zookeeper Leader Port: 3888
- Zookeeper Election Port (Peer port): 2888
- Broker: 9092
- REST Proxy: 8082
- Schema Registry: 8081
- KSQL: 8088


- Tests
3:22 ,
