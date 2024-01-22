# Consistency and Consensus

As we saw with the _eventual consistency_ of asynchronous replication schemes, there are no guarantees as to when consistency will actually be achieved. There are ways to mitigate this, but sometimes we value strong consistency guarantees over performance and fault tolerance.

Furthermore, an important abstraction that many distributed systems rely on is the ability to agree on something, or to come to a _consensus_. The key is to be able to do this effectively in the face of unreliable networks and failures.
