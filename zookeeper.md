#### Why is Zookeeper necessary for Apache Kafka?

- **Controller election**
  - The controller is one of the most important broking entity in a Kafka ecosystem, and it also has the responsibility to maintain the leader-follower relationship across all the partitions. If a node by some reason is shutting down, it’s the controller’s responsibility to tell all the replicas to act as partition leaders in order to fulfill the duties of the partition leaders on the node that is about to fail. So, whenever a node shuts down, a new controller can be elected and it can also be made sure that at any given time, there is only one controller and all the follower nodes have agreed on that.
- **Configuration Of Topics**
  - The configuration regarding all the topics including the list of existing topics, the number of partitions for each topic, the location of all the replicas, list of configuration overrides for all topics and which node is the preferred leader, etc.
- **Access control lists**
  - Access control lists or ACLs for all the topics are also maintained within Zookeeper.
- **Membership of the cluster**
  - Zookeeper also maintains a list of all the brokers that are functioning at any given moment and are a part of the cluster.

#### Default ports
- Zookeeper Client Port: 2181
- Zookeeper Leader Port: 3888
- Zookeeper Election Port (Peer port): 2888
