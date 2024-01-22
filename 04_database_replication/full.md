# Database Replication

Database replication is the process of creating copies of a database and storing them across various on-premise or cloud destinations.

It provides several key benefits:

- Better geographic locality for a global user base, since replicas can be placed closer geographically to the users that read from them
- Fault tolerance via redundancy
- Better read/write throughput since we split the load across multiple replicas

There are two types of replication:

- **Synchronous replication:** Whenever a new write comes into the system, we need to wait for the write to propagate across all nodes before it can be deemed successful and allow other transactions.
  - Slow but guarantees strong consistency
- **Asynchronous replication:** Writes might not need to entirely propagate through the system before we start other transactions.
  - Enables faster write throughput but sacrifices consistency (Eventual consistency)

## Improving Eventual Consistency

Eventual consistency allows us to reap the performance benefit of not having to wait for writes to completely propagate through our system before we can do anything else. However, there are some issues.

One such case is if a user makes a write, but reads the updated value before the write is propagated to the replica that they're reading from, they might read stale data and think the application is broken or slow. One way to mitigate this is to **read your own writes**:

- Whenever you write to a database replica, read from the same replica for some X time.
- Set X based on how long it takes to propagate that write to other replicas so that once X has passed, we can lift the restriction.

Another issue we could run into is if we read from replicas that are progressively less updated, resulting in reading data "out of order".

For example, let's say we're trying to get the latest event that occurred in an event log, and events 1, 2, and 3 occur in that order. The ordering of those events isn't guaranteed to be preserved while propagating across replica nodes. We could read from a replica that's seen all 3 events, but then read from another replica that's only seen the first 2. The result is we display event 2 as being later than event 3, which is incorrect.

A possible solution for this is to have each user always read data from the same replica. This guarantees that our reads are **monotonic reads** - we might still read stale data from the same replica, but at least it will be in the correct order.

## Single Leader Replication

In a single leader replication (sometimes referred to as Master-Slave or Active-Passive), we designate a specific replica node in the database cluster as a leader and write to that node only, having it manage the responsibility of propagating writes to other nodes.

This guarantees that we won’t have any write conflicts since all writes are processed by only one node. However, this also means we have a single point of failure (the leader) and slower write throughput since all writes can only go through a single node.

### Possible Failure Scenarios

In single leader replication, follower failures are pretty easy to recover from since the leader can just update the follower after it comes back online. Specifically, the leader can see what the follower’s last write was prior to failure in the replication log and backfill accordingly.

However, leader failures can result in many issues:

- The leader might actually be up, but the follower’s unable to connect due to network issues, which would result in it thinking it needs to promote itself to be the new leader
- A failure might result in lost writes if the leader was in the middle of propagating new writes to followers.
- When a leader comes back online after a new leader has already been determined, we could end up with two leaders propagating conflicting writes as clients send writes to both nodes (Split brain).

In general, single leader replication makes sense in situations where workloads are read-heavy rather than write-heavy. We can offload reads to multiple follower nodes and have the leader node focus on writes.

### Single Leader Replication in the Wild

- Most relational databases like MySQL and PostgreSQL use single-leader replication. However, some NoSQL databases like MongoDB, AWS DynamoDB, and RethinkDB support it as well
- Some distributed message brokers like [RabbitMQ](https://blog.rabbitmq.com/posts/2020/07/disaster-recovery-and-high-availability-101/#data-redundancy-tools---back-ups-and-replication) and [Kafka](https://cwiki.apache.org/confluence/display/kafka/kafka+replication#KafkaReplication-Datareplication) (which we'll talk about when we get to batch and stream processing), also support leader-based replication to provide high availability

## Multi-Leader Replication

In multi-leader replication, we write to multiple leader nodes instead of just one. This provides some redundancy compared to single-leader replication, as well as higher write throughput. It especially makes sense, performance wise, in situations where data needs to be available in multiple regions (each region should have its own leader).

There are a few topologies for organizing our write propagation flow between leaders.

### Topologies

#### Circle Topology

As the name suggests, leader nodes are arranged in a circular fashion, with each leader node passing writes to the next leader node in the circle

If a single node in the circle fails, the previous node in the circle that was passing writes no longer knows what to do. Hence, fault tolerance is non-existent in this topology

#### Star Topology

In a star topology, we designate a central leader node, which outer nodes pass their writes to. The central node then propagates these writes to the other nodes

If the outer nodes die, we're fine since the central node can continue to communicate with the other remaining outer nodes, so it's a little bit more fault tolerant than the Circle Topology. But if the central leader node dies, then we're screwed

#### All-to-all Topology

An all-to-all topology is a "complete graph" structure where every node propagates writes to every other node in the system (every node is the "central" node from the Star Topology)

This is even more fault tolerant than the star topology since now if any node dies, the rest of the nodes can still communicate with each other. However, there are still some issues with this toplogy:

- Since writes are being propagated from every node to every other node, there could be cases where duplicate writes get propagated out
- Writes might not necessarily be in order, which presents an issue if we have causally dependent writes (for example, write B modifies a row created by write A)

There are some ways to mitigate these issues. We can fix the duplicates issue by keeping track in our Replication Log which nodes have seen a given write

### Solutions for Write Conflicts

Multi leader replication could result in concurrent writes that are unaware of each other, causing inconsistency in the database (write conflicts). There are a few solutions for mitigating this

#### Conflict Avoidance

As the name implies, conflict avoidance just has us avoid conflicts altogether by having writes for a particular key only go to one replica. This limits our write throughput, so it's not ideal

#### Last Write Wins

In a last-write wins conflict resolution strategy, we use the timestamp of the write to determine what the value of a key should be. The write with the latest timestamp wins

Determining what timestamp to use can be tricky - for one thing, which timestamp do we trust? Sender timestamps are unreliable since clients can spoof their timestamp to be years in the future and ensure their write always wins.

Receiver timestamps, surprisingly, can also be unreliable. Computers rely on quartz crystals which vibrate at a specific frequency to determine the time. Due to factors like weather conditions and natural degredation, these frequencies can change. This results in computers having slightly different clocks over time, a process known as **clock skew**

- There are some ways to mitigate clock skew, such as using Network Time Protocol (NTP) to get a more accurate timestamp from a time server using a GPS clock.
- However this solution isn't perfect since we're subject to network delays if we're making requests to servers

#### Version Vector

A version vector is just an array that contains the number of writes a given node has seen from every other node. For example, `[1, 3, 2]` represents "1 write from partition 1, 3 writes from partition 2, and 2 writes from partition 3"

We then use this vector to either merge data together and resolve the conflict or store sibling data and offload conflict resolution to the client. For example, if we have `[1, 1, 2]` vs. `[2, 2, 2]`, we know the second version vector is more up to date

#### CRDT (Conflict-Free Replicated Data Types)

CRDTs are special data structures that allow the database to easily merge data types to resolve write conflicts. Some examples are a counter or a set.

- **Operational CRDTs**: Database nodes send operations to each other to keep their data in sync (e.g., sending increment operations to keep a distributed counter in sync).
  - Not idempotent, so don’t do well when we have duplicate or dropped requests.
  - Furthermore, if there are causally dependent operations, we might run into some trouble.
- **State-based CRDTs**: Database nodes send the entire CRDT itself.
  - These can be propagated through the system via the Gossip Protocol, in which nodes in the cluster send their CRDTs to some other random nodes (which may have already seen the message).
  - CRDTs can grow large, which may result in a lot of data being sent over the network.

### Multi-Leader replication in the wild

- CRDTs are used by systems like Redis (an in-memory cache) and Riak (a multi-leader/leaderless distributed key-value store).

## Leaderless Replication

Leaderless replication forgoes designating specific nodes as leader nodes entirely. In leaderless replication, we can write to any node and read from any node. This means we have high availability and fault tolerance, since every node is effectively a leader node. It also gives us high read AND write throughput.

### Quorums

To guarantee that we have the most recent values whenever we read from the system, we need a **quorum**, which just means "majority"

Writers write to a majority of nodes so that readers can guarantee that at least one of their return values will be the most recent when they read from a majority of nodes. In mathematical terms:

```
W (# of nodes we write to) + R (# of nodes we read from) > N (# of total nodes)
```

A nifty trick we can also do with quorums is **read repair**. Whenever we read values from R nodes, we might see that some of the results are out of date. We can then write the updated value back to their respective nodes.

There are some issues with quorums:

- We can still have cases in which writes arrive in different orders to a majority of nodes, causing disagreement amongst them as to which one is actually the most recent.
- Writes could also just fail, violating that inequality condition we just defined.

#### Sloppy Quorums

Let's imagine a client is able to talk to _some_ database nodes during a network interruption, but not _all_ the nodes it needs to assemble a quorum. We have two options here:

1. Return errors for all requests for which we can't reach a quorum of nodes
2. Accept writes anyways, but write them to nodes that _are_ reachable, but which aren't necessarily the nodes that we normally write to.

The 2nd option causes a _sloppy quorum_ where the _W_ and _R_ in our inequality aren't among the designated _N_ "home" nodes. For example, if replication nodes for the US region fail, we could establish a quorum using some nodes from the EU region instead.

Once the original home nodes come back up, we need to propagate the writes that were sent to those temporary writer nodes back to those home nodes. This process is called _hinted handoff_.

### Anti-Entropy

Another way to prevent stale reads is to propagate writes in the background between nodes. For example, if node A has writes 1, 2, 3, 4, 5, and node B only has writes 2, 3, 5, we'd need the first node to send writes 1 and 4 over.

One way to do this is to just send the entire replication log with all the writes from node A. But this would be inefficient since all we need is the diff (just writes 1 and 4).

We can quickly obtain this diff using a **Merkle Tree**, which is a tree of hashes computed over data rows. Each individual row gets hashed to a value, and those values are combined and hashed hierarchically until we get a root hash over all the rows.

Using a binary tree search, we can efficiently identify what's changed in a dataset by comparing hash values. For example, the root hash will tell us if there is any change across our entire data set, and we can examine child hashes recursively to track down which specific rows have changed.

### Leaderless Replication in the Wild

- Amazon's [Dynamo paper](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) popularized the leaderless replication architecture. So systems that were inspired by it support leaderless replication:
  - Apache Cassandra
  - Riak
  - [Voldemort](https://www.project-voldemort.com/voldemort/), a distributed key-value store designed by LinkedIn for high scalability
- Interestingly, AWS DynamoDB does _NOT_ use leaderless replication despite having _Dynamo_ in the name

## Additional Reading / Material

- _Designing Data-Intensive Applications_, Chapter 5, "Replication"
- **jordanhasnolife System Design 2.0 Playlist**:
  - ["Intro to Replication"](https://www.youtube.com/watch?v=FIPCDRRBGz4&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=16)
  - ["Dealing with Stale Reads"](https://www.youtube.com/watch?v=Y29yuEoBmjM&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=17)
  - ["Single Leader Replication"](https://www.youtube.com/watch?v=8h-a7TsXw28&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=18)
  - ["Multi Leader Replication"](https://www.youtube.com/watch?v=tffuvQtiTwY&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=19)
  - ["Dealing with Write Conflicts"](https://www.youtube.com/watch?v=sa4BJAFT8sU&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=20)
  - ["CRDTs"](https://www.youtube.com/watch?v=FG5Varj1Ows&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=21)
  - ["Leaderless Replication"](https://www.youtube.com/watch?v=Jy4Cm2WEZVg&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=22)
  - ["Quorums"](https://www.youtube.com/watch?v=DAONthD50g0&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=23)
- **EnjoyAlgorithms System Design Blog**:
  - ["Introduction to Database Replication"](https://www.enjoyalgorithms.com/blog/introduction-to-database-replication-system-design)
  - ["Master-Slave Replication (Single Leader Replication)"](https://www.enjoyalgorithms.com/blog/master-slave-replication-databases)
