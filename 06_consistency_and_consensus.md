# Consistency and Consensus

As we saw with the _eventual consistency_ of asynchronous replication schemes, there are no guarantees as to when consistency will actually be achieved. There are ways to mitigate this, but sometimes we value strong consistency guarantees over performance and fault tolerance.

Furthermore, an important abstraction that many distributed systems rely on is the ability to agree on something, or to come to a _consensus_. The key is to be able to do this effectively in the face of unreliable networks and failures.

## Two-Phase Commit

Two Phase Commit is a way of performing distributed writes or writes across multiple partitions while guaranteeing the atomicity of transactions (writes either succeed on every partition or fail on every partition).

Writing to different partitions is unlike the propagating writes across replicas because a single transaction to multiple partitions is effectively one logical write split up across multiple nodes. Hence this write needs to be atomic.

Two phase commit first designates a coordinator node to orchestrate the process. Afterwards, sending requests to every partition node to initiate the process, the following occurs:

- Each partition looks at its write-ahead log and determines whether it is able to add the write or not.
  - If yes, it will grab the locks for the affected rows and send back a success response before waiting for further action.
  - If no, it will send back a failure response.
- If all partitions send back a success response, the coordinator node writes in its commit log that the transaction should be committed across all nodes.
  - In the event of a failure at this point, the coordinator could read back from the commit log to retry the process.
- After doing this, the coordinator will tell all partition nodes to commit, and they all do so locally and respond accordingly when completed.

There are a few issues with Two Phase Commit:

- Single point of failure, the coordinator node could go down and cause the partition nodes to hold their locks, preventing other writes until the coordinator comes back online.
- If a receiver node goes down after the commit point, the coordinator node will have to send a request to commit repeatedly until the receiver comes back online.
- Basically, we want to avoid 2PC whenever possible because it's slow and hard. So try to avoid writing across multiple partitions if possible.

## Linearizable Storage

Linearizable storage dictates that whenever we perform a read, we can never go back in time (we always read the most recent thing). In other words, we want to preserve the ordering of writes

This is important for determining who is the most recent person to obtain a lock in a distributed locking system or who is the current leader in a single leader database replication setup.

Coordination services like Zookeeper will need this to keep track of database replication leaders and the latest status of application servers, for example.

Let's take a look at how linearizable storage applies to different replication setups:

**Single-leader replication setup**

In single leader replication, log writes are guaranteed to be in order since we only have one writer node keeping track of everything. That seems like that would produce linearizable storage, right?

Turns out, not necessarily! If the client reads from the leader before it can replicate a new write over to a follower, and then goes down, that will force the client to read from the follower with an outdated value. So this isn’t actually linearizable storage.

**Multi-leader or leaderless replication setup**

In multi-leader or leaderless replication, writes can go to many places and we could have concurrent writes, so we can’t make any guarantees about maintaining a chronological ordering (or it doesn’t even make sense to). We should try to achieve a _consistent_ ordering across all nodes instead

There are a couple of ways we might do this:

**Version vectors**

- If we see a higher number of writes across all partitions for a given node, we can determine that version vector to be more recent.
- We can use some arbitrary mechanism for tie-breakers (interleaved version vector numbers). For example, we just pick the one with the higher left-most partition writes as being more recent.
- Version vectors take O(N) space where N is the number of replicas.

**Lamport clocks**

- A counter stored across every replica node which gets incremented on every write, both on the client AND the database.
- Whenever we read the counter value on the client side, we write back the max of the database counter and the client counter alongside the write.
  - This value overrides the one on both the client and server, guaranteeing that all writes following this one will have a higher counter value.
- When sorting writes, we sort by the counter. If they’re the same, we can sort by node number.
- Unfortunately, we could still end up READING old data if replication hasn’t propagated through the system, so this isn’t actually linearizable storage. We only get ordering of writes after the fact.

## Distributed Consensus

The **Raft distributed consensus protocol** provides a way to build a distributed log that’s linearizable.

### Raft Leader Election

Below is the process for electing a new leader in Raft:

- One node is designated as a leader and sends heartbeats to all of its followers.
- If a heartbeat is not received after some randomized number of seconds (to prevent simultaneous complaints), a follower node will initiate an election.
- Follower node proposes itself as a candidate for the election corresponding to the next term.
  - If the leader comes back online, it will change itself to a follower for the next term and vote “yes”.
- The election is held as follows:

  - A follower from a previous term number (which is determined by the latest write that it’s seen) will change itself to a follower of the proposed term number and vote “yes”.
  - A follower who has a write from a newer term number will vote “no” and also update its own term number value to the one being proposed if it is higher.

- If a candidate gets “yes” votes from a quorum of nodes, it wins the election and becomes the new leader.

### Raft Writes

In Raft, we designate a leader that handles all writes. These writes are recorded in the form of logs, which contain an operation and a term number. Given the fact that only one node handles writes, it's possible that followers could have out of date logs.

Raft has 2 invariants when it comes to writes to address out of date logs:

1. There is only one leader per term
2. Successful writes must make the log fully up to date.

So writes backfill logs in addition to writing new values in order to keep them up to date. Furthermore, this invariant guarantees that if two logs are the same at a given point in time, every entry prior to that point will be the same

How does this actually work in practice? Whenever a new write arrives to the leader, the leader will broadcast it to the follower nodes along with the write it has at its latest index. Then the following occurs:

- If a follower node does not have the write at the current index, it will respond with a failure message.
  - The leader will decrement its index and try again and again until it matches on an index that the follower does have the write for.
  - At that point, the follower will backfill its own log and respond with a success message.
- Otherwise, the follower will just update its log and respond with a success message.
- Once the leader hears “yes” from a quorum of nodes, it will tell everybody to commit.

In conclusion, Raft is fault-tolerant and creates linearizable storage; however, it is slow and shouldn’t be used except in very specific situations where you need write correctness.

## Additional Reading / Material

- _Designing Data-Intensive Applications_, Chapter 9, "Consistency and Consensus"
- **jordanhasnolife System Design 2.0 Playlist**:
  - ["Two Phase Commit"](https://www.youtube.com/watch?v=7DoT2sTGulc&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=26)
  - ["Linearizable Databases"](https://www.youtube.com/watch?v=C_XLEeWUq3M&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=28)
  - ["Distributed Consensus - Raft Leader Election"](https://www.youtube.com/watch?v=Al2JNJBGG30&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=29)
  - ["Distributed Consensus - Raft Writes"](https://www.youtube.com/watch?v=FByzF2D_-KU&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=30)
