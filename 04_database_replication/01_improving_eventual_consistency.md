# Improving Eventual Consistency

Eventual consistency allows us to reap the performance benefit of not having to wait for writes to completely propagate through our system before we can do anything else. However, there are some issues.

One such case is if a user makes a write, but reads the updated value before the write is propagated to the replica that they're reading from, they might read stale data and think the application is broken or slow.

![stale-read](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/stale-read-eventual-consistency.png?alt=media&token=1fba4669-e342-4f21-b065-ab405d60fd0d)

One way to mitigate this is to **read your own writes**:

- Whenever you write to a database replica, read from the same replica for some X time.
- Set X based on how long it takes to propagate that write to other replicas so that once X has passed, we can lift the restriction.

![read-your-own-writes](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/read-your-own-writes.png?alt=media&token=e78195e9-4b41-463e-b2d2-9a5596626371)

Another issue we could run into is if we read from replicas that are progressively less updated, resulting in reading data "out of order". Let's look at an example:

![out-of-order](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/out-of-order-reads.png?alt=media&token=1c00a0c1-83c7-4026-9e32-1c5d5d11742b)

A possible solution for this is to have each user always read data from the same replica. This guarantees that our reads are **monotonic reads** - we might still read stale data from the same replica, but at least it will be in the correct order.
