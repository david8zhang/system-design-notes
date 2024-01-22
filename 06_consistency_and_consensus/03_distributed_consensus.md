# Distributed Consensus

The **Raft distributed consensus protocol** provides a way to build a distributed log that’s linearizable.

## Raft Leader Election

Below is the process for electing a new leader in Raft:

- One node is designated as a leader and sends heartbeats to all of its followers.
- If a heartbeat is not received after some randomized number of seconds (to prevent simultaneous complaints), a follower node will initiate an election.
- Follower node proposes itself as a candidate for the election corresponding to the next term.
  - If the leader comes back online, it will change itself to a follower for the next term and vote “yes”.
- The election is held as follows:

  - A follower from a previous term number (which is determined by the latest write that it’s seen) will change itself to a follower of the proposed term number and vote “yes”.
  - A follower who has a write from a newer term number will vote “no” and also update its own term number value to the one being proposed if it is higher.

- If a candidate gets “yes” votes from a quorum of nodes, it wins the election and becomes the new leader.

## Raft Writes

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
