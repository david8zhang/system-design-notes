# Single Leader Replication

In a single leader replication (sometimes referred to as Master-Slave or Active-Passive), we designate a specific replica node in the database cluster as a leader and write to that node only, having it manage the responsibility of propagating writes to other nodes.

This guarantees that we won’t have any write conflicts since all writes are processed by only one node. However, this also means we have a single point of failure (the leader) and slower write throughput since all writes can only go through a single node.

## Possible Failure Scenarios

In single leader replication, follower failures are pretty easy to recover from since the leader can just update the follower after it comes back online. Specifically, the leader can see what the follower’s last write was prior to failure in the replication log and backfill accordingly.

However, leader failures can result in many issues:

- The leader might actually be up, but the follower’s unable to connect due to network issues, which would result in it thinking it needs to promote itself to be the new leader
- A failure might result in lost writes if the leader was in the middle of propagating new writes to followers.
- When a leader comes back online after a new leader has already been determined, we could end up with two leaders propagating conflicting writes as clients send writes to both nodes (Split brain).

In general, single leader replication makes sense in situations where workloads are read-heavy rather than write-heavy. We can offload reads to multiple follower nodes and have the leader node focus on writes.

## Single Leader Replication in the Wild

- Most relational databases like MySQL and PostgreSQL use single-leader replication. However, some NoSQL databases like MongoDB, AWS DynamoDB, and RethinkDB support it as well
- Some distributed message brokers like [RabbitMQ](https://blog.rabbitmq.com/posts/2020/07/disaster-recovery-and-high-availability-101/#data-redundancy-tools---back-ups-and-replication) and [Kafka](https://cwiki.apache.org/confluence/display/kafka/kafka+replication#KafkaReplication-Datareplication) (which we'll talk about when we get to batch and stream processing), also support leader-based replication to provide high availability
