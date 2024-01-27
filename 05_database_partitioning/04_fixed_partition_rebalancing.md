# Fixed Partition Rebalancing

We can also define a fixed number of partitions we have across our entire system rather than tying the number of partitions to the number of available nodes.

For example, if we have 4 partitions across 3 nodes (12 total), if a node goes down we can divide the 4 that are orphaned across the remaining 2 nodes. Additionally, if we were to gain a node in our system, we could extract 3 partitions out and place them in that node. We utilize our system resources more effectively by distributing partitions to nodes according to their hardware capacity and performance - give more of the orphaned partitions to the more powerful machines.

A downside that might occur with this approach is that as our database grows, the size of each partition will grow in turn, since the total number of partitions is static. So that means we'd need to pick a good fixed partition number from the outset: one that isn't too large so as to make recovery from node failures expensive and complex, but also one that isn't too small so as to incur too much overhead.
