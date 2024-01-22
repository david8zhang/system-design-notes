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
