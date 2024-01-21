# Database Indexes

A database index is a data structure that allows you to efficiently search for specific records in your database. Below are a few database index types to know.

## Hash Index

In a hash index, we pass each database key into a hash function and store the key at a memory address corresponding to the hash. This gives us extremely optimized reads and writes, as hash tables provide constant time lookup and storage.

However, hash indexes are limited to small datasets since the hash of the key might not fit within the memory address space.

### Hash Indexes in the Wild

- [Bitcask](https://docs.riak.com/riak/kv/2.2.3/setup/planning/backend/bitcask/index.html), the default storage engine in [Riak](https://riak.com/index.html), a NoSQL key-value data store, uses Hash Indexes

## LSM Tree + SSTable

LSM trees are tree data structures used in write-optimized databases. "LSM" stands for “Log Structured Merge Tree” and "SSTable" stands for “Sorted String Table”.

In general, they're optimized for write throughput since we’re writing to a data structure in memory (fast) before writing to disk (slow). In addition, the sorted nature of SSTables allow us to write _sequentially_ to disk which is much faster than writing randomly.

### SSTable Serialization

As mentioned before, LSM Trees are auto-balancing binary search trees (e.g. Red-Black trees) that we insert database keys into. Once it gets to be a certain size, we flush the LSM tree to disk as an SSTable. We do a tree traversal to preserve the sorted ordering of keys

Every time we do this SSTable serialization, we create a brand new SSTable which might store new values for keys that exist in previous SSTables. We never _delete_ values from SSTables, instead we store a marker indicating that the value was deleted called a "tombstone"

- This means that reading might be slow since we’d need to scan every SSTable if the key doesn’t exist in our current LSM Tree
- However, we can merge SSTables together in a background process called SSTable compaction (a process similar to the "merge" operation in the mergesort algorithm) reducing the number of SSTables we need to search through.

### Write-Ahead Logs

A write-ahead log (WAL) is just a log of all the write operations we are doing whenever we insert keys into the tree. We maintain a write-ahead log on disk to maintain the durability of the LSM tree.

- _Durability_ means "survivability in the event of failures". For example, if somebody trips on a power cord and wipes our LSM tree in memory, we can recover using the operations we recorded in our WAL

### LSM Tree + SSTables in the Wild

The following systems all use LSM Trees (sometimes referred to as a _memtable_) and SSTables

- [Apache Cassandra](https://cassandra.apache.org/doc/latest/cassandra/architecture/storage-engine.html), an open source NoSQL database
- [Apache HBase](https://hbase.apache.org/), the "Hadoop database", an open source, non-relational big data store
- [LevelDB](https://github.com/google/leveldb/blob/main/doc/impl.md), an open source key-value storage library developed by Google
- [RocksDB](https://rocksdb.org/), a high performance embedded key-value store, which is actually a fork of LevelDB developed and maintained by Facebook

**Note**: All of the above systems _store_ data as LSM Trees and SSTables rather than _index_ data in that way.

## B Tree / B+ Trees

B Tree indexes are database indexes that utilize a self-balancing N-ary search tree known as a B tree, and are the most widely used type of index. Unlike LSM trees and SSTables, B tree indexes are stored mostly on disk.

### B Tree Properties

B Trees have a special property in that each node contains a range of keys in sorted order, and there is a lower and upper bound on the number of keys and children that a node may have:

- These bounds are usually determined by the size of a page on disk
- Inserting into a node might trigger a cascading series of splits to preserve the balance of the binary tree, which may incur some CPU penalties. However, the advantages we get in terms of read efficiency are typically worth these downsides

Reading from B Trees is generally fast due to the high branching factor (large number of children at each node). To be more specific, since an internal B tree node can have more than just 2 children, the logarithmic time complexity will have a higher base than 2

### B+ Trees

B+ Trees are similar to B Trees, except:

- Whereas B Trees can store data at the interior node level, B+ Trees only store data at the leaf node level
- Leaf nodes are linked together (every leaf node has a reference to the next leaf node sequentially).

The key advantage offered by this setup is that a full scan of all objects in a tree requires just one linear pass through all leaf nodes, as opposed to having to do a full tree traversal. The disadvantage is that since only leaf nodes contain data, accessing data might take longer since you have to go deeper in the tree

### B Trees / B+ Trees in the Wild

B Trees are used by most relational databases, for example:

- [InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-physical-structure.html), the storage engine of MySQL, an open source relational DBMS
- [Microsoft SQL Server](https://learn.microsoft.com/en-us/sql/relational-databases/indexes/indexes?view=sql-server-ver16), a proprietary DBMS developed and maintained by Microsoft
- [PostgreSQL](https://www.postgresql.org/docs/current/btree-implementation.html), another open source relational DBMS

However, some non-relational databases use them as well, such as

- [MongoDB](https://www.mongodb.com/docs/manual/indexes/), a document oriented NoSQL database
- [Oracle NoSQL](https://www.oracle.com/database/nosql/technologies/nosql/), a NoSQL, distributed key-value database from Oracle

## Additional Reading / Material

- _Designing Data-Intensive Applications_, Chapter 3: "Storage and Retrieval", section 1: "Data Structures that Power Your Database"
- **jordanhasnolife System Design 2.0 Playlist**:
  - ["Database Indexes"](https://www.youtube.com/watch?v=LzNdvuj3a5M&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=2)
  - ["Hash Indexes"](https://www.youtube.com/watch?v=I1wQsY-Nh_k&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=3)
  - ["B Tree Indexes"](https://www.youtube.com/watch?v=Z2OaqmxiH20&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=4)
  - ["LSM Tree + SSTable Indexes"](https://www.youtube.com/watch?v=ciGAVER_erw&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=5)
- **ByteByteGo System Design Video Series**: ["The Secret Sauce Behind No SQL"](https://www.youtube.com/watch?v=I6jB0nM9SKU)
