# Linearizable Storage

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
