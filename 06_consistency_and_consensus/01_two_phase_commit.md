# Two-Phase Commit

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
