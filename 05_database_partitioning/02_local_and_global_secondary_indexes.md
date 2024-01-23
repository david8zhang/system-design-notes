# Local / Global Secondary Indexes

**Local secondary indexes** are indexes local to a specific partition on a particular column.

- Writing is fairly straightforward - every time we write a value to a partition, we also write it into the index on the same node.
- Reading is slower since we need to scan through the local secondary index of every partition and stitch together the result.

**Global secondary indexes** are indexes over the entire dataset, split up across our partitions.

- Reading becomes much faster since we don’t need to query every single partition’s index. We can hone in on just the partitions that store whatever range we’re looking for.
- Writes will become slower since we might end up saving a key on two different partitions if the indexed location for that key is not in the same partition as its hash.
