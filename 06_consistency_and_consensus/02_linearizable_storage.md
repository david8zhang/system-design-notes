# Linearizable Storage

Linearizable storage dictates that whenever we perform a read, we can never go back in time (we always read the most recent thing). In other words, we want to preserve the ordering of writes. This is important for determining who is the most recent person to obtain a lock in a distributed locking system or who is the current leader in a single leader database replication setup.

Coordination services like Zookeeper will need this to keep track of database replication leaders and the latest status of application servers, for example.

Let's take a look at how linearizable storage applies to different replication setups:

## Single-leader replication setup

In single leader replication, log writes are guaranteed to be in order since we only have one writer node keeping track of everything. That seems like that would produce linearizable storage, right?

Turns out, not necessarily! If the client reads from the leader before it can replicate a new write over to a follower, and then goes down, that will force the client to read from the follower with an outdated value. So this isn’t actually linearizable storage.

## Multi-leader or leaderless replication setup

In multi-leader or leaderless replication, writes can go to many places and we could have concurrent writes, so we can’t make any guarantees about maintaining a chronological ordering (or it doesn’t even make sense to). We should try to achieve a _consistent_ ordering across all nodes instead

There are a couple of ways we might do this:

**Version vectors**

If we see a higher number of writes across all partitions for a given node, we can determine that version vector to be more recent. For example `[1, 2, 1]` is more recent than `[0, 1, 0]`

We can use some arbitrary mechanism for tie-breakers (interleaved version vector numbers). For example, we just pick the one with the higher left-most partition writes as being more recent.

Version vectors take O(N) space where N is the number of replicas, so it might not be the most efficient solution if we have a lot of replicas.

**Lamport clocks**

A Lamport clock is a counter stored across every replica node which gets incremented on every write, both on the client AND the database.

Whenever we read the counter value on the client side, we write back the max of the database counter and the client counter alongside the write. This value overrides the one on both the client and server, guaranteeing that all writes following this one will have a higher counter value. When sorting writes, we sort by the counter. If they’re the same, we can sort by node number.

Unfortunately, we could still end up READING old data if replication hasn’t propagated through the system, so this isn’t actually linearizable storage. We only get ordering of writes after the fact.

## CAP Theorem and PACELC Theorem

CAP Theorem states that it's impossible for a distributed system to provide all three of the following:

- **Consistency**: The same response is given to all identical requests. This is synonymous with _linearizability_, in which we provide a consistent ordering of writes such that all client will read data in the same order regardless of which partition they're reading from. (Cruciallly it's NOT the same thing as Consistency in [ACID](/topic/03_ACID-transactions))
- **Availability**: The system is still accessible during partial failures
- **Partition Tolerance**: Operations remain intact when nodes are unavailable.

Historically, CAP Theorem has been used to analyze database technologies and the tradeoffs associated with them. For example:

- "MongoDB's a CP database! its single leader replication setup enables it to provide consistent write ordering, but could experience downtime due to leader failures.".
- "Cassandra is an AP database! It has a leaderless or multi-leader replication setup that provides greater fault tolerance, but is eventually consistent in the face of write conflicts."

However, some have criticized CAP Theorem of being too simple (most notably, Martin Kleppmann, author of _Designing Data-Intensive Applications_).

Thus, the _PACELC theorem_ attempts to address some of its shortcomings, preferring to frame database systems as either leaning towards latency sensitivity or strong consistency. it states:

> "In the case of a network partition (P), one has to choose between either Availability (A) and Consistency (C), but else (E), even when the system is running normally in the absence of partitions, one has to choose between latency (L) and loss of consistency (C)"
