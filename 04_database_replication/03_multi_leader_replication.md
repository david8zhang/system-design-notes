# Multi-Leader Replication

In multi-leader replication, we write to multiple leader nodes instead of just one. This provides some redundancy compared to single-leader replication, as well as higher write throughput. It especially makes sense, performance wise, in situations where data needs to be available in multiple regions (each region should have its own leader).

There are a few topologies for organizing our write propagation flow between leaders.

## Topologies

### Circle Topology

As the name suggests, leader nodes are arranged in a circular fashion, with each leader node passing writes to the next leader node in the circle

![circle-topology](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/multi-leader-circle.png?alt=media&token=78ac3e44-a87f-44a7-bd8d-6af0bdb8f56c)

If a single node in the circle fails, the previous node in the circle that was passing writes no longer knows what to do. Hence, fault tolerance is non-existent in this topology

### Star Topology

In a star topology, we designate a central leader node, which outer nodes pass their writes to. The central node then propagates these writes to the other nodes

![star-topology](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/multi-leader-star.png?alt=media&token=e3e4aa81-e50d-4c03-9d31-8bd923404e86)

If the outer nodes die, we're fine since the central node can continue to communicate with the other remaining outer nodes, so it's a little bit more fault tolerant than the Circle Topology. But if the central leader node dies, then we're screwed

### All-to-all Topology

An all-to-all topology is a "complete graph" structure where every node propagates writes to every other node in the system (every node is the "central" node from the Star Topology)

![all-to-all-topology](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/multi-leader-all-to-all.png?alt=media&token=8fe6e9c8-39dc-4419-9596-707ca70707d2)

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

![last-write-wins](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/multi-leader-lww.png?alt=media&token=c8f56197-4ec0-4a56-9d7f-1c4168d15f6b)

Determining what timestamp to use can be tricky - for one thing, which timestamp do we trust? Sender timestamps are unreliable since clients can spoof their timestamp to be years in the future and ensure their write always wins.

Receiver timestamps, surprisingly, can also be unreliable. Computers rely on quartz crystals which vibrate at a specific frequency to determine the time. Due to factors like weather conditions and natural degredation, these frequencies can change. This results in computers having slightly different clocks over time, a process known as **clock skew**

- There are some ways to mitigate clock skew, such as using Network Time Protocol (NTP) to get a more accurate timestamp from a time server using a GPS clock.
- However this solution isn't perfect since we're subject to network delays if we're making requests to servers

### Version Vector

A version vector is just an array that contains the number of writes a given node has seen from every other node. For example, `[1, 3, 2]` represents "1 write from partition 1, 3 writes from partition 2, and 2 writes from partition 3"

We then use this vector to either merge data together and resolve the conflict or store sibling data and offload conflict resolution to the client. Let's look at an example:

![version-vectors](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/multi-leader-version-vectors.png?alt=media&token=e19a9e6c-adbb-4330-899d-5431f90d5c5c)

### CRDT (Conflict-Free Replicated Data Types)

CRDTs are special data structures that allow the database to easily merge data types to resolve write conflicts. Some examples are a counter or a set.

#### Operational CRDTs

Database nodes send operations to each other to keep their data in sync. These have some latency benefits compared to state-based CRDTs since we don't need to send as much data over the network.

![operational-crdt](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/operational-crdt.png?alt=media&token=bd01ad15-6b75-4ef5-8686-54b319321687)

However, there are a few problems with Operational CRDTs. For one thing, they're not _idempotent_, so don’t do well when we have duplicate or dropped requests. For example in the distributed counter above, if we sent that increment operation multiple times due to some kind of failure / retry mechanism, we could have some issues.

Furthermore, what if we have _causally dependent_ operations? For example:

![operational-crdt-causal-dep](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/operational-crdt-causal-dep.png?alt=media&token=1492f730-ae84-4082-91d6-c8890e505e75)

#### State-based CRDTs

Database nodes send the entire CRDT itself, and the nodes "merge" states together to update their states. The "merge" logic must be:

- Associative: `f(a, f(b, c)) = f(f(a, b), c)`
- Commutative: `f(a, b) = f(b, a)`
- Idempotent: `f(a, b) = f(f(a, b), b)`

Let's take a look at an example.

![state-crdt](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/state-crdt.png?alt=media&token=a72a6438-b43f-488c-8102-5c1b94aa5630)

We can see that Node 1 sending that set multiple times would result in the same merged result on Node 2. This idempotency also enables state CRDTs to be propagated via the _Gossip Protocol_, in which nodes in the cluster send their CRDTs to some other random nodes (which may have already seen the message).

![gossip-protocol](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/gossip-protocol.png?alt=media&token=df30061a-e938-4a6b-9bb8-36e4de2d8924)

### Multi-Leader replication in the wild

- CRDTs are used by systems like Redis (an in-memory cache) and Riak (a multi-leader/leaderless distributed key-value store).
