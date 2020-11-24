## Stream processing


#### Stream Processing Topology

- stream
  - A stream is the most important abstraction provided by Kafka Streams: it represents an unbounded, continuously updating data set. A stream is an ordered, replayable, and fault-tolerant sequence of immutable data records, where a data record is defined as a key-value pair.
- stream processing application
  - any program that makes use of the Kafka Streams library. It defines its computational logic through one or more processor topologies, where a processor topology is a graph of stream processors (nodes) that are connected by streams (edges).
- stream processor
  - A stream processor is a node in the processor topology; it represents a processing step to transform data in streams by receiving one input record at a time from its upstream processors in the topology, applying its operation to it, and may subsequently produce one or more output records to its downstream processors.
  - There are two special processors in the topology:
    - **Source Processor** : A source processor is a special type of stream processor that does not have any upstream processors. It produces an input stream to its topology from one or multiple Kafka topics by consuming records from these topics and forwarding them to its down-stream processors.
    - **Sink Processor** : A sink processor is a special type of stream processor that does not have down-stream processors. It sends any received records from its up-stream processors to a specified Kafka topic.

[](images/streamprocessor_topology.png)


- APIs
  - **Kafka Streams DSL**  : provides the most common data transformation operations such as map, filter, join and aggregations out of the box
  - **Lower-level Processor API** : allows developers define and connect custom processors as well as to interact with state stores.

- Time
  - **Event time** - The point in time when an event or data record occurred, i.e. was originally created "at the source".
  - **Processing time** - The point in time when the event or data record happens to be processed by the stream processing application, i.e. when the record is being consumed.
  - **Ingestion time** - The point in time when an event or data record is stored in a topic partition by a Kafka broker.

- Aggregations
  - An aggregation operation takes one input stream or table, and yields a new table by combining multiple input records into a single output record. Examples of aggregations are computing counts or sum.
  - In the Kafka Streams DSL, an input stream of an aggregation can be a KStream or a KTable, but the output stream will always be a KTable
  -  This allows Kafka Streams to update an aggregate value upon the out-of-order arrival of further records after the value was produced and emitted. When such out-of-order arrival happens, the aggregating KStream or KTable emits a new aggregate value. Because the output is a KTable, the new value is considered to overwrite the old value with the same key in subsequent processing steps.

- Windowing
  - Windowing lets you control how to `group records that have the same key` for stateful operations such as aggregations or joins into so-called windows. Windows are tracked per record key.
  - **grace period** : When working with windows, you can specify a grace period for the window. This grace period controls how long Kafka Streams will wait for out-of-order data records for a given window. If a record arrives after the grace period of a window has passed, the record is discarded and will not be processed in that window. Specifically, a record is discarded if its timestamp dictates it belongs to a window, but the current stream time is greater than the end of the window plus the grace period.

  - Types :
    - Tumbling time windows
    - Hopping time windows
    - Sliding time windows
    - Session Windows


- Duality of Streams and Tables
  - When implementing stream processing use cases in practice, you typically need both streams and also databases. An example use case that is very common in practice is an e-commerce application that enriches an incoming stream of customer transactions with the latest customer information from a database table. In other words, streams are everywhere, but databases are everywhere, too.
  - first-class support for streams and tables

- States
  - Some stream processing applications don't require state, which means the processing of a message is independent from the processing of all other messages. However, being able to maintain state opens up many possibilities for sophisticated stream processing applications: you can join input streams, or group and aggregate data records.
  - Kafka Streams provides so-called state stores, which can be used by stream processing applications to store and query data. This is an important capability when implementing stateful operations. Every task in Kafka Streams embeds one or more state stores that can be accessed via APIs to store and query data required for processing. These state stores can either be a persistent key-value store, an in-memory hashmap, or another convenient data structure. Kafka Streams offers fault-tolerance and automatic recovery for local state stores.
  - Kafka Streams allows direct read-only queries of the state stores by methods, threads, processes or applications external to the stream processing application that created the state stores. This is provided through a feature called Interactive Queries. All stores are named and Interactive Queries exposes only the read operations of the underlying implementation.

- PROCESSING GUARANTEES


- Out-of-Order Handling


- Transform a stream
  - Stateless transformations
  - Stateful transformations
- Aggregating
  - An aggregation operation takes one input stream or table, and yields a new table by combining multiple input records into a single output record.
  - In the Kafka Streams DSL, an input stream of an aggregation can be a KStream or a KTable, but the output stream will always be a KTable

- Joining
  - Join co-partitioning requirements
  - KStream-KStream Join
  - KTable-KTable Join
  - KStream-KTable Join
  - KStream-GlobalKTable Join


- Why does co-partitioning of two Kstreams in kafka require same number of partitions for both the streams?
 - As the name "co-partition" indicates, you want to put data from different topic but same key to the same Kafka Streams application instance. If you don't have the same number of partitions, it's not possible to get this behavior.
 - Assume you have topic A with 2 partitions and topic B with 3 partitions. Thus, it can happen that one record with key X is hashed to partitions A-0 and B-1 (ie, not same partition number). However, for a different key Y it might be hashed to A-0 but B-2.
 - Only if the number of partitions is the same for both topics, records with same key end up in the same partitions (of different topics of course), and this allows to process A-0/B-0 and A-1/B-1 etc together.
 - https://medium.com/xebia-france/kafka-streams-co-partitioning-requirements-illustrated-2033f686b19c


- Elastic scaling of application
  - https://kafka.apache.org/26/documentation/streams/developer-guide/running-app#elastic-scaling-of-your-application

- Kafka Streams work allocation
  - Kafka Streams uses the concepts of partitions and tasks as logical units of its parallelism model based on Kafka topic partitions. There are close links between Kafka Streams and Kafka in the context of parallelism:
    - Each stream partition is a totally ordered sequence of data records and maps to a Kafka topic partition.
    - A data record in the stream maps to a Kafka message from that topic.
    - The keys of data records determine the partitioning of data in both Kafka and Kafka Streams, i.e., how data is routed to specific partitions within topics.
  - An application's processor topology is scaled by breaking it into multiple tasks
  - More specifically, Kafka Streams creates a fixed number of tasks based on the input stream partitions for the application, with each task assigned a list of partitions from the input streams (i.e., Kafka topics). The assignment of partitions to tasks never changes so that each task is a fixed unit of parallelism of the application. Tasks can then instantiate their own processor topology based on the assigned partitions; they also maintain a buffer for each of its assigned partitions and process messages one-at-a-time from these record buffers. As a result stream tasks can be processed independently and in parallel without manual intervention.
  - Slightly simplified, the maximum parallelism at which your application may run is bounded by the maximum number of stream tasks, which itself is determined by maximum number of partitions of the input topic(s) the application is reading from
  - Threading Model
    - Kafka Streams allows the user to configure the number of threads that the library can use to parallelize processing within an application instance.
    - Each thread can execute one or more tasks with their processor topologies independently


#### Stateless Operators

|Type |Transformation
-|-
Branch| KStream → KStream[]
Filter|KStream → KStream, KTable → KTable
Inverse Filter|KStream → KStream, KTable → KTable
FlatMap|KStream → KStream
FlatMap (values only)|KStream → KStream
Foreach|KStream → void , KStream → void, KTable → void
GroupByKey|KStream → KGroupedStream
GroupBy|KStream → KGroupedStream, KTable → KGroupedTable
Cogroup|KGroupedStream → CogroupedKStream, CogroupedKStream → CogroupedKStream
Map|KStream → KStream
Map(Values only)|KStream → KStream, KTable → KTable
Merge|KStream → KStream
Peek|KStream → KStream
Print|KStream → void
SelectKey|KStream → KStream
Table to Stream|KTable → KStream
Stream to Table|KStream → KTable
Repartition|KStream → KStream


#### Stateful Operators

|Type|Transformation
-|-
Aggregate|KGroupedStream → KTable, KGroupedTable → KTable
Aggregate (windowed)|KGroupedStream → KTable
Count|KGroupedStream → KTable, KGroupedTable → KTable
Count (windowed)|KGroupedStream → KTable
Reduce|KGroupedStream → KTable, KGroupedTable → KTable
Reduce (windowed)|KGroupedStream → KTable

#### Joins
Join operands|	Type|	(INNER) JOIN|	LEFT JOIN|	OUTER JOIN
-|-|-|-|-
KStream-to-KStream|	Windowed|	Supported|	Supported|	Supported
KTable-to-KTable|	Non-windowed|	Supported|	Supported|	Supported
KTable-to-KTable(Foreign-Key Join)|	Non-windowed|	Supported|	Supported	|Not Supported
KStream-to-KTable|	Non-windowed|	Supported|	Supported|	Not Supported
KStream-to-GlobalKTable|	Non-windowed|	Supported|	Supported|	Not Supported
KTable-to-GlobalKTable|	N/A|	Not Supported	|Not Supported	|Not Supported


#### Windowing

Window name	|Behavior|	Short description|
-|-|-
Hopping time window	|Time-based	|Fixed-size, overlapping windows (wall timestamp)
Tumbling time window|	Time-based|	Fixed-size, non-overlapping, gap-less windows
Sliding time window	|Time-based	|Fixed-size, overlapping windows that work on differences between record timestamps
Session window|	Session-based|	Dynamically-sized, non-overlapping, data-driven windows


#### Resources
  - https://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api
  - https://medium.com/@andy.bryant/kafka-streams-work-allocation-4f31c24753cc
  - https://www.michael-noll.com/blog/2018/04/05/of-stream-and-tables-in-kafka-and-stream-processing-part1/
  - https://www.confluent.io/blog/kafka-streams-tables-part-1-event-streaming/

#### KIPs
  - https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Streams
  - https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Streams+Internal+Data+Management
  -
