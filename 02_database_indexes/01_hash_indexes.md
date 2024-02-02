# Hash Index

In a hash index, we pass each database key into a hash function and store the value at a memory address corresponding to the hash. This gives us extremely optimized reads and writes, as hash tables provide constant time lookup and storage

![Example of a hash index](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/01_hash_indexes.png?alt=media&token=f2c44559-f222-4746-b0ae-c5fccffad810)

However, hash indexes are limited to small datasets since the hash of the key might not fit within the memory address space. Though you could implement a hash index on disk, it's not efficient to perform random I/O. Furthermore, they're not particularly good for range queries.

## Hash Indexes in the Wild

- [Bitcask](https://docs.riak.com/riak/kv/2.2.3/setup/planning/backend/bitcask/index.html), the default storage engine in [Riak](https://riak.com/index.html), a NoSQL key-value data store, uses Hash Indexes
