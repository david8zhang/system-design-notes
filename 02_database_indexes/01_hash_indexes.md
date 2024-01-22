# Hash Index

In a hash index, we pass each database key into a hash function and store the key at a memory address corresponding to the hash. This gives us extremely optimized reads and writes, as hash tables provide constant time lookup and storage.

However, hash indexes are limited to small datasets since the hash of the key might not fit within the memory address space.

## Hash Indexes in the Wild

- [Bitcask](https://docs.riak.com/riak/kv/2.2.3/setup/planning/backend/bitcask/index.html), the default storage engine in [Riak](https://riak.com/index.html), a NoSQL key-value data store, uses Hash Indexes
