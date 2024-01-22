# Multi-Leader Replication

In multi-leader replication, we write to multiple leader nodes instead of just one. This provides some redundancy compared to single-leader replication, as well as higher write throughput. It especially makes sense, performance wise, in situations where data needs to be available in multiple regions (each region should have its own leader).

There are a few topologies for organizing our write propagation flow between leaders.

## Topologies

### Circle Topology

As the name suggests, leader nodes are arranged in a circular fashion, with each leader node passing writes to the next leader node in the circle

If a single node in the circle fails, the previous node in the circle that was passing writes no longer knows what to do. Hence, fault tolerance is non-existent in this topology

### Star Topology

In a star topology, we designate a central leader node, which outer nodes pass their writes to. The central node then propagates these writes to the other nodes

If the outer nodes die, we're fine since the central node can continue to communicate with the other remaining outer nodes, so it's a little bit more fault tolerant than the Circle Topology. But if the central leader node dies, then we're screwed

### All-to-all Topology

An all-to-all topology is a "complete graph" structure where every node propagates writes to every other node in the system (every node is the "central" node from the Star Topology)

This is even more fault tolerant than the star topology since now if any node dies, the rest of the nodes can still communicate with each other. However, there are still some issues with this toplogy:

- Since writes are being propagated from every node to every other node, there could be cases where duplicate writes get propagated out
- Writes might not necessarily be in order, which presents an issue if we have causally dependent writes (for example, write B modifies a row created by write A)

There are some ways to mitigate these issues. We can fix the duplicates issue by keeping track in our Replication Log which nodes have seen a given write

### Solutions for Write Conflicts

Multi leader replication could result in concurrent writes that are unaware of each other, causing inconsistency in the database (write conflicts). There are a few solutions for mitigating this

### Conflict Avoidance

As the name implies, conflict avoidance just has us avoid conflicts altogether by having writes for a particular key only go to one replica. This limits our write throughput, so it's not ideal

### Last Write Wins

In a last-write wins conflict resolution strategy, we use the timestamp of the write to determine what the value of a key should be. The write with the latest timestamp wins

Determining what timestamp to use can be tricky - for one thing, which timestamp do we trust? Sender timestamps are unreliable since clients can spoof their timestamp to be years in the future and ensure their write always wins.

Receiver timestamps, surprisingly, can also be unreliable. Computers rely on quartz crystals which vibrate at a specific frequency to determine the time. Due to factors like weather conditions and natural degredation, these frequencies can change. This results in computers having slightly different clocks over time, a process known as **clock skew**

- There are some ways to mitigate clock skew, such as using Network Time Protocol (NTP) to get a more accurate timestamp from a time server using a GPS clock.
- However this solution isn't perfect since we're subject to network delays if we're making requests to servers

### Version Vector

A version vector is just an array that contains the number of writes a given node has seen from every other node. For example, `[1, 3, 2]` represents "1 write from partition 1, 3 writes from partition 2, and 2 writes from partition 3"

We then use this vector to either merge data together and resolve the conflict or store sibling data and offload conflict resolution to the client. For example, if we have `[1, 1, 2]` vs. `[2, 2, 2]`, we know the second version vector is more up to date

### CRDT (Conflict-Free Replicated Data Types)

CRDTs are special data structures that allow the database to easily merge data types to resolve write conflicts. Some examples are a counter or a set.

- **Operational CRDTs**: Database nodes send operations to each other to keep their data in sync (e.g., sending increment operations to keep a distributed counter in sync).
  - Not idempotent, so don’t do well when we have duplicate or dropped requests.
  - Furthermore, if there are causally dependent operations, we might run into some trouble.
- **State-based CRDTs**: Database nodes send the entire CRDT itself.
  - These can be propagated through the system via the Gossip Protocol, in which nodes in the cluster send their CRDTs to some other random nodes (which may have already seen the message).
  - CRDTs can grow large, which may result in a lot of data being sent over the network.

### Multi-Leader replication in the wild

- CRDTs are used by systems like Redis (an in-memory cache) and Riak (a multi-leader/leaderless distributed key-value store).
