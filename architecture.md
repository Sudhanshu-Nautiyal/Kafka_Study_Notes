### Introduction

- Event streaming
  - event streaming is the practice of capturing data in real-time from event sources like databases, sensors, mobile devices, cloud services, and software applications in the form of streams of events; storing these event streams durably for later retrieval; manipulating, processing, and reacting to the event streams in real-time as well as retrospectively; and routing the event streams to different destination technologies as needed
  - kafka is an event streaming platform
  - Kafka combines three key capabilities so you can implement your use cases for event streaming end-to-end with a single battle-tested solution:
    - To **publish (write)** and **subscribe to (read)** streams of events, including continuous import/export of your data from other systems.
    - To **store** streams of events durably and reliably for as long as you want.
    - To **process** streams of events as they occur or retrospectively.

- Event Sourcing
  - Event sourcing is a style of application design where state changes are logged as a time-ordered sequence of records. Kafka's support for very large stored log data makes it an excellent backend for an application built in this style.

- Commit Log
  - Kafka can serve as a kind of external commit-log for a distributed system. The log helps replicate data between nodes and acts as a re-syncing mechanism for failed nodes to restore their data. The log compaction feature in Kafka helps support this usage


#### Kafka in nutshell
- Key terminologies
  - Servers
  - Clients
  - Event
  - Topics
  - Producer
  - Consumer

#### Kafka APIs
- Admin API
- Producer API
- Consumer API
- Kafka Streams API
- Kafka Connect API


#### Design
- Don't fear the filesystem!
  - The key fact about disk performance is that the throughput of hard drives has been diverging from the latency of a disk seek for the last decade. As a result the performance of linear writes on a JBOD configuration with six 7200rpm SATA RAID-5 array is about 600MB/sec but the performance of random writes is only about 100k/sec—a difference of over 6000X. These linear reads and writes are the most predictable of all usage patterns, and are heavily optimized by the operating system. A modern operating system provides read-ahead and write-behind techniques that prefetch data in large block multiples and group smaller logical writes into large physical writes
  - in-memory cache vs relying on OS pagecache
- Constant Time Suffices
  - BTrees are the most versatile data structure available, and make it possible to support a wide variety of transactional and non-transactional semantics in the messaging system. They do come with a fairly high cost, though: Btree operations are O(log N). Normally O(log N) is considered essentially equivalent to constant time, but this is not true for disk operations.
  - Intuitively a persistent queue could be built on simple reads and appends to files as is commonly the case with logging solutions. This structure has the advantage that all operations are O(1) and reads do not block writes or each other
  - Having access to virtually unlimited disk space without any performance penalty means that we can provide some features not usually found in a messaging system. For example, in Kafka, instead of attempting to delete messages as soon as they are consumed, we can retain messages for a relatively long period (say a week). This leads to a great deal of flexibility for consumers, as we will describe.
- Efficiency
  - Problem : too many small I/O operations
    - The small I/O problem happens both between the client and the server and in the server's own persistent operations.
    - To avoid this, our protocol is built around a "message set" abstraction that naturally groups messages together. This allows network requests to group messages together and amortize the overhead of the network roundtrip rather than sending a single message at a time. The server in turn appends chunks of messages to its log in one go, and the consumer fetches large linear chunks at a time.
    - Batching leads to larger network packets, larger sequential disk operations, contiguous memory blocks, and so on, all of which allows Kafka to turn a bursty stream of random message writes into linear writes that flow to the consumers.
  - Problem : excessive byte copying
    - At low message rates this is not an issue, but under load the impact is significant. To avoid this we employ a standardized binary message format that is shared by the producer, the broker, and the consumer (so data chunks can be transferred without modification between them).
    - The message log maintained by the broker is itself just a directory of files, each populated by a sequence of message sets that have been written to disk in the same format used by the producer and consumer. Maintaining this common format allows optimization of the most important operation: network transfer of persistent log chunks. Modern unix operating systems offer a highly optimized code path for transferring data out of pagecache to a socket; in Linux this is done with the sendfile system call.
    - sendfile and zero-copy
- End-to-end Batch Compression
  - In some cases the bottleneck is actually not CPU or disk but network bandwidth
  - Efficient compression requires compressing multiple messages together rather than compressing each message individually.
  - Kafka supports this with an efficient batching format. A batch of messages can be clumped together compressed and sent to the server in this form. This batch of messages will be written in compressed form and will remain compressed in the log and will only be decompressed by the consumer.
  - Kafka supports GZIP, Snappy, LZ4 and ZStandard compression protocols.
- Push vs. pull
  - Kafka follows a more traditional design, shared by most messaging systems, where data is pushed to the broker from the producer and pulled from the broker by the consumer.
  - Disadvantage of push based consumers
    - consumer tends to be overwhelmed when its rate of consumption falls below the rate of production
    - A pull-based system has the nicer property that the consumer simply falls behind and catches up when it can

- Consumer Position

- Offline Data Load

- Static Membership


- Message Delivery Semantics
  - At most once—Messages may be lost but are never redelivered.
  - At least once—Messages are never lost but may be redelivered.
  - Exactly once—this is what people actually want, each message is delivered once and only once.

- Replication
  - Kafka replicates the log for each topic's partitions across a configurable number of servers
  - an set this replication factor on a topic-by-topic basis
  - This allows automatic failover to these replicas when a server in the cluster fails so messages remain available in the presence of failures.
  - The unit of replication is the topic partition. Under non-failure conditions, each partition in Kafka has a single leader and zero or more followers
  - The total number of replicas including the leader constitute the replication factor
  - All reads and writes go to the leader of the partition
  - Followers consume messages from the leader just as a normal Kafka consumer would and apply them to their own log.
  - Kafka node liveness has two conditions
    - A node must be able to maintain its session with ZooKeeper (via ZooKeeper's heartbeat mechanism)
    - If it is a follower it must replicate the writes happening on the leader and not fall "too far" behind
  - We refer to nodes satisfying these two conditions as being **"in sync"** to avoid the vagueness of "alive" or "failed".
  - The leader keeps track of the set of "in sync" nodes
  - If a follower dies, gets stuck, or falls behind, the leader will remove it from the list of in sync replicas. The determination of stuck and lagging replicas is controlled by the replica.lag.time.max.ms configuration.
  - a message is considered committed when all in sync replicas for that partition have applied it to their log
  - Only committed messages are ever given out to the consumer.
  -  Producers, on the other hand, have the option of either waiting for the message to be committed or not, depending on their preference for tradeoff between latency and durability.This preference is controlled by the acks setting that the producer uses
  - Note that topics have a setting for the "minimum number" of in-sync replicas that is checked when the producer requests acknowledgment that a message has been written to the full set of in-sync replicas
  - The guarantee that Kafka offers is that a committed message will not be lost, as long as there is at least one in sync replica alive, at all times.

- Replicated Logs
  - Quorum
  - ISR
  - Leader election
    - Instead of majority vote, Kafka dynamically maintains a set of in-sync replicas (ISR) that are caught-up to the leader. Only members of this set are eligible for election as leader.
    - A write to a Kafka partition is not considered committed until all in-sync replicas have received the write. This ISR set is persisted to ZooKeeper whenever it changes. Because of this, any replica in the ISR is eligible to be elected leader. This is an important factor for Kafka's usage model where there are many partitions and ensuring leadership balance is important. With this ISR model and f+1 replicas, a Kafka topic can tolerate f failures without losing committed messages.
  - Unclean leader election

- Availability and Durability Guarantees
  - Topic-level configurations that can be used to prefer message durability over availability:
    - **Disable unclean leader election** - if all replicas become unavailable, then the partition will remain unavailable until the most recent leader becomes available again. This effectively prefers unavailability over the risk of message loss. See the previous section on Unclean Leader Election for clarification.
    - **Specify a minimum ISR size** - the partition will only accept writes if the size of the ISR is above a certain minimum, in order to prevent the loss of messages that were written to just a single replica, which subsequently becomes unavailable. This setting only takes effect if the producer uses acks=all and guarantees that the message will be acknowledged by at least this many in-sync replicas. This setting offers a trade-off between consistency and availability. A higher setting for minimum ISR size guarantees better consistency since the message is guaranteed to be written to more replicas which reduces the probability that it will be lost. However, it reduces availability since the partition will be unavailable for writes if the number of in-sync replicas drops below the minimum threshold.

- Replica Management
  - Kafka cluster will manage hundreds or thousands of these partitions
  - Kafka attempt to balance partitions within a cluster in a round-robin fashion to avoid clustering all partitions for high-volume topics on a small number of nodes
  - Likewise it try to balance leadership so that each node is the leader for a proportional share of its partitions.
  - It is also important to optimize the leadership election process as that is the critical window of unavailability. A naive implementation of leader election would end up running an election per partition for all partitions a node hosted when that node failed. Instead, we elect one of the brokers as the "controller". This controller detects failures at the broker level and is responsible for changing the leader of all affected partitions in a failed broker. The result is that we are able to batch together many of the required leadership change notifications which makes the election process far cheaper and faster for a large number of partitions. If the controller fails, one of the surviving brokers will become the new controller.

- Log Compaction
  - Log compaction ensures that Kafka will always retain at least the last known value for each message key within the log of data for a single topic partition

- The Log Cleaner


- Quotas
  - Network Bandwidth Quotas
  - Request Rate Quotas
  - Enforcement
