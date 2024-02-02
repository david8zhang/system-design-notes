# LSM Tree + SSTable

LSM trees are tree data structures used in write-optimized databases. "LSM" stands for “Log Structured Merge Tree” and "SSTable" stands for “Sorted String Table”.

In general, they're optimized for write throughput since we’re writing to a data structure in memory (fast) before writing to disk (slow). In addition, the sorted nature of SSTables allow us to write _sequentially_ to disk which is much faster than writing randomly.

## SSTable Serialization

As mentioned before, LSM Trees are auto-balancing binary search trees (e.g. Red-Black trees) that we insert database keys into. Once it gets to be a certain size, we flush the LSM tree to disk as an SSTable. We do a tree traversal to preserve the sorted ordering of keys

Every time we do this SSTable serialization, we create a brand new SSTable which might store new values for keys that exist in previous SSTables. We never _delete_ values from SSTables, instead we store a marker indicating that the value was deleted called a "tombstone"

- This means that reading might be slow since we’d need to scan every SSTable if the key doesn’t exist in our current LSM Tree
- However, we can merge SSTables together in a background process called SSTable compaction (a process similar to the "merge" operation in the mergesort algorithm) reducing the number of SSTables we need to search through.

Here's an example for how this process works:

![LSM Tree, Compaction](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/02_lsm_tree_sstable.png?alt=media&token=37508996-3f57-4acc-a7a6-92a47fb6752c)

## Write-Ahead Logs

A write-ahead log (WAL) is just a log of all the write operations we are doing whenever we insert keys into the tree. We maintain a write-ahead log on disk to maintain the durability of the LSM tree.

- _Durability_ means "survivability in the event of failures". For example, if somebody trips on a power cord and wipes our LSM tree in memory, we can recover using the operations we recorded in our WAL

## LSM Tree + SSTables in the Wild

The following systems all use LSM Trees (sometimes referred to as a _memtable_) and SSTables

- [Apache Cassandra](https://cassandra.apache.org/doc/latest/cassandra/architecture/storage-engine.html), an open source NoSQL database
- [Apache HBase](https://hbase.apache.org/), the "Hadoop database", an open source, non-relational big data store
- [LevelDB](https://github.com/google/leveldb/blob/main/doc/impl.md), an open source key-value storage library developed by Google
- [RocksDB](https://rocksdb.org/), a high performance embedded key-value store, which is actually a fork of LevelDB developed and maintained by Facebook

**Note**: All of the above systems _store_ data as LSM Trees and SSTables rather than _index_ data in that way.
