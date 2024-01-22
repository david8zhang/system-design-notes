# Fixed Partition Rebalancing

We can also define a fixed number of partitions we have across our entire system rather than tying the number of partitions to the number of available nodes.

For example, if we have 4 partitions across 3 nodes (12 total), if a node goes down we can divide the 4 that are orphaned across the remaining 2 nodes. Additionally, if we were to gain a node in our system, we could extract 3 partitions out and place them in that node.
