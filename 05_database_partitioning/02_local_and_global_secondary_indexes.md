# Local / Global Secondary Indexes

A _secondary index_ is an additional index that's stored alongside your primary index, which might keep data in a different sort order to improve the performance of certain other queries which might not benefit from the sort order of the primary index. There are two main types of secondary indexes.

**Local secondary indexes** are indexes local to a specific partition on a particular column.

![local-secondary-index](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/local-secondary-index.png?alt=media&token=5e6a97dd-88f3-447e-82a1-a81c373a0be3)

- Writing is fairly straightforward - every time we write a value to a partition, we also write it into the index on the same node.
- Reading is slower since we need to scan through the local secondary index of every partition and stitch together the result.

**Global secondary indexes** are indexes over the entire dataset, split up across our partitions.

![global-secondary-index](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/global-secondary-index.png?alt=media&token=20af19b5-caa7-4109-a1a5-92429e00d6ad)

- Reading becomes much faster since we don’t need to query every single partition’s index. We can hone in on just the partitions that store whatever range we’re looking for.
- Writes will become slower since we might end up saving a key on two different partitions if the indexed location for that key is not in the same partition as its hash.
