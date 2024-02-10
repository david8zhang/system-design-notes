# Fixed Partition Rebalancing

We can also define a fixed number of partitions we have across our entire system rather than tying the number of partitions to the number of available nodes.

![fixed-partition](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/fixed-partitions.png?alt=media&token=d3c12cd2-3fda-4b02-beca-607bd2b989ca)

We can also utilize our system resources more effectively by distributing partitions to nodes according to their hardware capacity and performance, e.g. give more of the orphaned partitions to the more powerful machines.

A downside that might occur with this approach is that as our database grows, the size of each partition will grow in turn, since the total number of partitions is static. So that means we'd need to pick a good fixed partition number from the outset: one that isn't too large so as to make recovery from node failures expensive and complex, but also one that isn't too small so as to incur too much overhead.
