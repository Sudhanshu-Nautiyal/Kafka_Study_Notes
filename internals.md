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
