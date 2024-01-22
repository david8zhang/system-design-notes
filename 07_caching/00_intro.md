# Caching

A cache is a hardware or software component that stores data so that future requests for that data can be served faster. In the context of an operating system, for example, we have CPU hardware caches set up hierarchically (L1, L2, L3) to speed up data access from main memory.

In distributed systems, caching provides several important benefits:

1. Reduce load on key components
2. Speed up reads and writes- caches typically store data on memory and are sometimes located physically closer to the client.

There are some challenges with caching - cache misses are expensive since we have to do a network request to the cache, search for the key and discover it’s missing, and then do a network request to the database. Furthermore, data consistency on caches is complex. If two clients each hit two different caches in two different regions, ensuring the data between the two is consistent can be a tough problem to solve.

### What data do we cache?

We generally want to cache the results of computationally expensive operations. These could include database query results or computations performed on data by application servers.

We can also cache popular static content such as HTML, images, or video files. (We'll talk more about this when we get to CDNs)

### Where does our cache live?

1. **Local on the application servers themselves**

   - This requires us to use consistent hashing in our load balancer to ensure requests go to the same server every time.
   - _Pros:_ Fewer network calls, very fast.
   - _Cons:_ Cache size is proportional to the number of servers.

2. **Global caching layer**
   - _Pros:_ Nodes in the global caching layer can be scaled independently.
   - _Cons:_ Extra network call, more can go wrong.

### Caches in the Wild

There are two popular choices for implementing application caches in software systems:

**Memcached**

[Memcached](https://memcached.org/) is an open source distributed in-memory store. It uses an LRU eviction policy as well as consistent hashing for partitioning data, meaning the requests to the same key will be sent to the same partition.

Memcached is fairly bare-bones and is more useful for a customized caching implementation involving multi-threading or leaderless/multi-leader replication.

**Redis**

[Redis](https://redis.io/) is a more feature-rich in memory store, with support for specialized data structures like hash maps, sorted sets, and geo-indexes. It uses a fixed number of partitions, re-partitioning using the gossip protocol. It also supports ACID transactions using its write ahead log and single threaded serial execution. Its replication strategy uses a single leader.
