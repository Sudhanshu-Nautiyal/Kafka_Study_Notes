#### Why is Zookeeper necessary for Apache Kafka?
- ZooKeeper servers will be deployed on multiple nodes. This is called an **ensemble**. An ensemble is a set of 2n + 1 ZooKeeper servers where n is any number greater than 0. The odd number of servers allows ZooKeeper to perform majority elections for leadership. At any given time, there can be up to n failed servers in an ensemble and the ZooKeeper cluster will keep quorum. If at any time, quorum is lost, the ZooKeeper cluster will go down.
- Zookeeper cluster to withstand loss of 2 server, require total of 2*2+1 = 5 servers.
- In Zookeeper multi-node configuration, initLimit and syncLimit are used to govern how long following ZooKeeper servers can take to initialize with the current leader and how long they can be out of sync with the leader. If tickTime=2000, initLimit=5 and syncLimit=2 then a follower can take (tickTimeinitLimit) = 10000ms to initialize and may be out of sync for up to (tickTimesyncLimit) = 4000ms

- In Zookeeper multi-node configuration, The server.* properties set the ensemble membership. The format is **server.$myid=$hostname:$leaderport:$electionport** , where:
  - **myid** is the server identification number. In this example, there are three servers, so each one will have a different myid with values 1, 2, and 3 respectively. The myid is set by creating a file named myid in the dataDir that contains a single integer in human readable ASCII text. This value must match one of the myid values from the configuration file. If another ensemble member has already been started with a conflicting myid value, an error will be thrown upon startup.
  - **leaderport** is used by followers to connect to the active leader. This port should be open between all ZooKeeper ensemble members.
  - **electionport** is used to perform leader elections between ensemble members. This port should be open between all ZooKeeper ensemble members.


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


#### Config

- tickTime
  - the basic time unit in milliseconds used by ZooKeeper. It is used to do heartbeats and the minimum session timeout will be twice the tickTime.
- initLimit
  - is timeouts ZooKeeper uses to limit the length of time the ZooKeeper servers in quorum have to connect to a leader
- syncLimit
  - limits how far out of date a server can be from a leader.
