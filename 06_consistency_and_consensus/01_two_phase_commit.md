# Two-Phase Commit

Two Phase Commit is a way of performing distributed writes or writes across multiple partitions while guaranteeing the atomicity of transactions (writes either succeed on every partition or fail on every partition).

Writing to different partitions is unlike propagating writes across replicas because a single transaction to multiple partitions is effectively one logical write split up across multiple nodes. Hence this write needs to be atomic.

Two-phase commit designates a _coordinator node_ to orchestrate the process. This node is usually just the application server performing the cross-partition write.

The following diagram describes the "happy case" flow for two-phase commit.

![two-phase-commit](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/2pc.png?alt=media&token=22a4e26f-4128-474b-8c2b-facdb19425bd)

A few issues can arise:

- If a node is unable to commit due to a conflict, the coordinator node will need to send an ABORT command to that node and stop the entire process.
- If the coordinator node goes down, all the partition nodes will hold onto their locks, preventing other writes until the coordinator comes back online.
- If a receiver node goes down after the commit point, the coordinator node will have to send a request to commit repeatedly until the receiver comes back online.

We generally want to avoid 2PC whenever possible because it's slow and hard. So try to avoid writing across multiple partitions if possible.
