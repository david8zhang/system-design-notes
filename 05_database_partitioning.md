# Database Partitioning

Once our database starts getting too large, we'll need to split the data up across different pieces (partitions). Complications will arise in terms of how we should best split the partition, identifying partitions that a query should be routed to, and re-partitioning in the event of failure.

For choosing partitions, we have some different options:

- **Range-based partitioning:** Divide our data into ranges (of timestamp or name in alphabetical order, for example).
  - Enables very fast range queries due to data locality.
  - Could result in hot spots if a particular range is accessed more often than another.
- **Hash-range based partitioning:** Hash the key and assign it to a partition corresponding to the hash.
  - Provides a more even distribution of keys, preventing hot spots.
  - Range queries are no longer as efficient since we don’t have data locality (but we can mitigate this using indexes).

## Functional partitioning (Federation)

Federation (or functional partitioning), splits up databases by function. An example of this is an e-commerce application like Amazon breaking up databases into users, reviews, and products rather than having one monolothic database that stores everything.

This gives us greater read and write throughput and minimizes replication lag. In addition, since datasets are smaller, we can cache more effectively since we can fit more of the dataset in memory. However, application logic will need to keep track of which database to read and write to and joins may end up being more complicated.

## Local / Global Secondary Indexes

**Local secondary indexes** are indexes local to a specific partition on a particular column.

- Writing is fairly straightforward - every time we write a value to a partition, we also write it into the index on the same node.
- Reading is slower since we need to scan through the local secondary index of every partition and stitch together the result.

**Global secondary indexes** are indexes over the entire dataset, split up across our partitions.

- Reading becomes much faster since we don’t need to query every single partition’s index. We can hone in on just the partitions that store whatever range we’re looking for.
- Writes will become slower since we might end up saving a key on two different partitions if the indexed location for that key is not in the same partition as its hash.

## Consistent Hashing

You might think a natural way to partition data is with the _modulo_ operator. If we have 3 partitions, for example, every key gets assigned to the partition corresponding to the hash of that key modulo 3.

But then what if one of the partitions goes down? Then we need to repartition across our whole dataset since now every key should be assigned to the partition corresponding to its hash modulo 2. This is obviously not ideal - we need some way to only rebalance the keys from the partition node that went down.

One way that we can accomplish just that is to use consistent hashing instead. In this scheme, we define hash ranges for every partition in such a way that whenever a partition goes down, we can extend the range for the remaining partitions to include the keys that were part of the partition that just went down.

In the event that the partition comes back online, we just reallocate those keys back to the partition in which they originally belonged.

## Fixed Partition Rebalancing

We can also define a fixed number of partitions we have across our entire system rather than tying the number of partitions to the number of available nodes.

For example, if we have 4 partitions across 3 nodes (12 total), if a node goes down we can divide the 4 that are orphaned across the remaining 2 nodes. Additionally, if we were to gain a node in our system, we could extract 3 partitions out and place them in that node.

## Additional Reading / Material

- _Designing Data-Intensive Applications_, Chapter 6, "Partitioning"
- **jordanhasnolife System Design 2.0 Playlist**:
  - ["Introduction to Partitioning"](https://www.youtube.com/watch?v=Bt8ZMC_Yuys&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=25)
  - ["Consistent Hashing"](https://www.youtube.com/watch?v=z-xxLoJAfmY&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=27)
- **Karan Pratap Singh's Open-source System Design Course**: ["Database Federation"](https://github.com/karanpratapsingh/system-design?tab=readme-ov-file#database-federation)
