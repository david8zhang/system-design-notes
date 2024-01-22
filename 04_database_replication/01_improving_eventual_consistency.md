# Improving Eventual Consistency

Eventual consistency allows us to reap the performance benefit of not having to wait for writes to completely propagate through our system before we can do anything else. However, there are some issues.

One such case is if a user makes a write, but reads the updated value before the write is propagated to the replica that they're reading from, they might read stale data and think the application is broken or slow. One way to mitigate this is to **read your own writes**:

- Whenever you write to a database replica, read from the same replica for some X time.
- Set X based on how long it takes to propagate that write to other replicas so that once X has passed, we can lift the restriction.

Another issue we could run into is if we read from replicas that are progressively less updated, resulting in reading data "out of order".

For example, let's say we're trying to get the latest event that occurred in an event log, and events 1, 2, and 3 occur in that order. The ordering of those events isn't guaranteed to be preserved while propagating across replica nodes. We could read from a replica that's seen all 3 events, but then read from another replica that's only seen the first 2. The result is we display event 2 as being later than event 3, which is incorrect.

A possible solution for this is to have each user always read data from the same replica. This guarantees that our reads are **monotonic reads** - we might still read stale data from the same replica, but at least it will be in the correct order.
