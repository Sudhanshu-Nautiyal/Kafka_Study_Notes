#### Cluster Membership

- Kafka uses Apache Zookeeper to maintain the list of brokers that are currently mem‐ bers of a cluster
- Every broker has a unique identifier that is either set in the broker configuration file or automatically generated. Every time a broker process starts, it registers itself with its ID in Zookeeper by creating an ephemeral node.
- Different Kafka components subscribe to the /brokers/ids path in Zookeeper where brokers are registered so they get notified when brokers are added or removed.
- When a broker is added or removed from the cluster , each watcher is notified

#### The Controller

- one of the kafka broker
- additional responsibility
  - electing partition leaders
  -  first broker that starts in the cluster becomes the controller by creating an ephemeral node in ZooKeeper called /controller
- Kafka uses Zookeeper’s ephemeral node feature to elect a controller and to notify the controller when nodes join and leave the cluster. The controller is responsible for electing leaders among the partitions and replicas whenever it notices nodes join and leave the cluster.
- Each time a controller is elected, it receives a new, higher controller epoch number through a Zookeeper con‐ ditional increment operation.
- The controller uses the epoch number to prevent a “split brain” scenario where two nodes believe each is the current controller.


#### Replication

- kafka is a distributed, partitioned, replicated commit log service
- There are two types of replicas:
  - **Leader replica**
    - Each partition has a single replica designated as the leader. All produce and con‐ sume requests go through the leader, in order to guarantee consistency.
  - **Follower replica**
    - All replicas for a partition that are not leaders are called followers. Followers don’t serve client requests; their only job is to replicate messages from the leader and stay up-to-date with the most recent messages the leader has.
    - In the event that a leader replica for a partition crashes, one of the follower replicas will be promoted to become the new leader for the partition.
    - In order to stay in sync with the leader, the replicas send the leader Fetch requests, the exact same type of requests that consumers send in order to consume messages.
    - Only in-sync replicas are eligible to be elected as partition leaders in case the existing leader fails.
    - The amount of time a follower can be inactive or behind before it is considered out of sync is controlled by the `replica.lag.time.max.ms` configuration parameter.
- Preferred leader
  - the replica that was the leader when the topic was originally created.
  -  It is preferred because when partitions are first created, the leaders are balanced between brokers
  - we expect that when the preferred leader is indeed the leader for all partitions in the cluster, load will be evenly balanced between brokers
  - By default, Kafka is configured with `auto.leader.rebalance.enable=true`, which will check if the preferred leader replica is not the current leader but is in-sync and trigger leader election to make the preferred leader the current leader.
  - The first replica in the replica list is always the preferred leader. This is true no matter who is the current leader and even if the replicas were reassigned to different brokers using the replica reassignment tool


#### Request Processing

- All requests have a standard header that includes:
  - Request type (also called API key)
  - Request version (so the brokers can handle clients of different versions and respond accordingly)
  - Correlation ID: a number that uniquely identifies the request and also appears in the response and in the error logs (the ID is used for troubleshooting)
  - Client ID: used to identify the application that sent the request

- acceptor thread
  - For each port the broker listens on, the broker runs an acceptor thread that creates a connection and hands it over to a processor thread for handling
- processor thread (network thread)
  - The network threads are responsible for taking requests from client connections, placing them in a request queue, and picking up responses from a response queue and sending them back to clients.
- request queue
  - Once requests are placed on the request queue, IO threads are responsible for picking them up and processing them. The most common types of requests are:
    - Produce requests : Sent by producers and contain messages the clients write to Kafka brokers.
    - Fetch requests : Sent by consumers and follower replicas when they read messages from Kafka brokers.
    - Metadata request : specifies which partitions exist in the topics, the replicas for each partition, and which replica is the leader. Metadata requests can be sent to any broker because all brokers have a metadata cache that contains this information.

#### kafka storage internals

- Topic
  - Topic can be thought of as being a container in which these partitions lie.
- Partition
  - Partitions are the units of storage in Kafka for messages
  - A partition, in theory, can be described as an immutable collection (or sequence) of messages
  - Kafka can only append messages to a partition
  - partition is tied to a broker , so unless a broker is leader or replica for a partition , that partition won't exist in a broker
- Offset
  - each kafka message in the partition log file gets has an ID also referred as offset
  - are sequential and incremental
- Segments
  - Each partition is sub-divided into segments.
  - A segment is simply a collection of messages of a partition. Instead of storing all the messages of a partition in a single file (think of the log file analogy again), Kafka splits them into chunks called segments. Doing this provides several advantages. Divide and Conquer FTW!
  - Most importantly, it makes purging data easy.
  - Kafka always writes the messages into these segment files under a partition
  - There is always an **active segment** to which Kafka writes to. Once the segment’s size limit is reached, a new segment file is created and that becomes the newly active segment.
  - Each segment file is created with the offset of the first message as its file name.
- Segment index
  - One of the common operations in Kafka is to read the message at a particular offset. For this, if it has to go to the log file to find the offset, it becomes an expensive task
  - The segment index maps offsets to their message’s position in the segment log.
  - The index file is memory mapped, and the offset look up uses binary search to find the nearest offset less than or equal to the target offset.

  
```
Kafka directory structure and files
$ tree test-topic*
test-topic-0
|-- 00000000000000000000.index
|-- 00000000000000000000.log
|-- 00000000000000000000.timeindex
`-- leader-epoch-checkpoint
test-topic-1
|-- 00000000000000000000.index
|-- 00000000000000000000.log
|-- 00000000000000000000.timeindex
`-- leader-epoch-checkpoint
test-topic-2
|-- 00000000000000000000.index
|-- 00000000000000000000.log
|-- 00000000000000000000.timeindex
`-- leader-epoch-checkpoint
```
