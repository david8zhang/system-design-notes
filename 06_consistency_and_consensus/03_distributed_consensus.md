# Distributed Consensus

The **Raft distributed consensus protocol** provides a way to build a distributed log that’s linearizable.

## Proposing a new leader in RAFT

Below is the process for electing a new leader in Raft:

One node is designated as a leader and sends heartbeats to all of its followers. If a heartbeat is not received after some randomized number of seconds (to prevent simultaneous complaints), a follower node will initiate an election.

Follower node proposes itself as a candidate for the election corresponding to the next term.

![raft-leader-proposal](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/RAFT-leader-proposal.png?alt=media&token=e4a51c5c-0845-48f4-8a07-2933bd12ca7f)

## Leader election

The election is held as follows:

- A follower from a previous term number (which is determined by the latest write that it’s seen) will change itself to a follower of the proposed term number and vote “yes”.
- A follower who has a write from a newer term number will vote “no” and also update its own term number value to the one being proposed if it is higher.
- If a candidate gets “yes” votes from a quorum of nodes, it wins the election and becomes the new leader.

![raft-leader-election](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/RAFT-leader-election.png?alt=media&token=8d5b5d44-270d-4327-a4e1-25c38ca05ea6)

## Raft Write Backfills

In Raft, we designate a leader that handles all writes. These writes are recorded in the form of logs, which contain an operation and a term number. Given the fact that only one node handles writes, it's possible that followers could have out of date logs.

Raft has 2 invariants when it comes to writes to address out of date logs:

1. There is only one leader per term
2. Successful writes must make the log fully up to date.

So writes backfill logs in addition to writing new values in order to keep them up to date. Furthermore, this invariant guarantees that if two logs are the same at a given point in time, every entry prior to that point will be the same

How does this actually work in practice? Let's look at a diagram of the process:

![raft-write-backfills](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/RAFT-write-backfills.png?alt=media&token=0bab4089-a7b7-47a1-9d82-4b2b7cd4bf3d)

In conclusion, Raft is fault-tolerant and creates linearizable storage; however, it is slow and shouldn’t be used except in very specific situations where you need write correctness.
