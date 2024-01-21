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

### Memcached

[Memcached](https://memcached.org/) is an open source distributed in-memory store. It uses an LRU eviction policy as well as consistent hashing for partitioning data, meaning the requests to the same key will be sent to the same partition.

Memcached is fairly bare-bones and is more useful for a customized caching implementation involving multi-threading or leaderless/multi-leader replication.

### Redis

[Redis](https://redis.io/) is a more feature-rich in memory store, with support for specialized data structures like hash maps, sorted sets, and geo-indexes. It uses a fixed number of partitions, re-partitioning using the gossip protocol. It also supports ACID transactions using its write ahead log and single threaded serial execution. Its replication strategy uses a single leader.

## Distributed Cache Writes

There are a few different methods for writing to a distributed cache.

### Write-Around Cache

A **write-around** caching strategy entails sending writes directly to the database, going "around" the cache. Of course, this means that the corresponding cached value will now be stale. We have a couple of ways to fix this:

1. Stale Read / TTL - We set a Time To Live (TTL) on the value in the cache, which is like an expiration date. Reads to the cache key will be stale until the TTL expires, at which point the value will be removed from the cache. A subsequent read on that key will go to the database, which will then repopulate the cache and set a new TTL

2. Invalidation - After writing to the database, we also invalidate its corresponding key in the cache. Subsequent reads will trigger the database to repopulate the cached value.

#### Pros and Cons of Write-Around

- _Pros:_ Database is the source of truth, and writes are simple since they're not that different from our usual, non-cache flow
- _Cons:_ Cache misses are expensive since we need to make 2 network calls, 1 for fetching the value from the cache and 1 for fetching the value from the database.

### Write-Through Cache

A **write-through** caching strategy is when we send writes to the cache, then proxy that request to the database. This means we may have inconsistent data between our cache and our database. Sometimes that isn't an issue, but in the cases where it is, we'd need to use two phase commit (2PC) to guarantee correctness.

#### Pros and Cons of Write-Through

- _Pros:_ Data is consistent between the cache and the database.
- _Cons:_ Correctness issues if we don’t use 2PC. If we do, 2PC is slow.

### Write-Back Cache

A **write-back** caching strategy is similar to the write-through caching strategy, except writes aren’t propagated to the database immediately. We do this mainly to optimize for lower write latency since we’re writing to the cache without having to also write to the database. The database write-back updates are performed asynchronously - at some point down the line, or maybe on a fixed interval, we group writes in the cache and send them to the database together.

There are a few problems that could occur - for one thing, if the cache just fails, then writes never go to the database and we have incorrect data. We can provide better fault tolerance through replication, though this may add a lot of complexity to the system.

Furthermore, if a user tries to read directly from the database before the write backs can happen, they may see a stale value. We can mitigate this with a distributed lock:

1. Whenever we write to the key in our cache, we also grab the distributed lock on that key.
2. We hold the lock until somebody tries to also grab that lock while reading that key in the database.
3. This other attempt to grab the lock would trigger a write back from the cache to the database, allowing the reader to see the up to date value.

Again, this adds a lot of complexity and latency, so we typically try to avoid needing to do this.

#### Pros and Cons of Write-Back

- _Pros:_ Low latency writes.
- _Cons:_ We may have stale or incorrect data

## Cache Eviction Policies

Our caches have limited storage space, so we'll need to figure out how best to manage what's stored in the cache and how to get rid of things. Our aim is to minimize cache misses, so let's take a look at how we can best do that.

### First-in First-out

The first value in the cache becomes the first value to be evicted. It's relatively simple to implement since it's basically a queue. Every time we add a new value to the cache, we delete whatever the oldest value in the cache was.

The major problem with this is that we don't consider data access patterns and might evict data that's still actively being queried. For example, even if many users are querying a specific key, that key will eventually become the oldest key added to the cache as new data is added. It will subsequently be evicted despite being the most popular key.

### Least Recently Used

A better policy for evicting data is the "Least Recently Used" (LRU) policy, the most commonly used eviction policy in practice. In LRU, the last _accessed_ key gets evicted, and more recently used keys get to stay in the cache. This solves the problem we saw earlier with First-in First-out - popular keys will remain cached since they will keep being used.

LRU is a bit more complicated to implement - rather than just having a queue we'd need to use a hashmap with a doubly linked list. We need a hashmap to identify the node to move to the head of the list, and a doubly-linked list to be able to shift things around in O(1) time.

### Least Frequently Used

A more sophisticated alternative is "Least Frequently Used", where we keep track of the _frequencies_ that a key is being accessed and evict according to that.

Using this policy alone also has some problems - if a key is referenced repeatedly for a short period of time and then not touched for a long time afterwards, it might stay in the cache simply because it has a high frequency count from that initial spike in popularity. In addition, new items in the cache might be removed too soon since they start with a low frequency counter.

Therefore, an LFU policy is typically best for situations in which access patterns of cached objects do not change often. For example, the Google logo will always be accessed at a fairly high rate, but a hype storm around a new logo for Instagram or Reddit might temporarily increase its recency in the cache. Once the hype dies down, those new logos will be evicted as their frequency count stops growing, while Google's remains in the cache.

## Content Delivery Networks

Content Delivery Networks (CDNs), are geographically distributed caches for static content, like HTML, image, video, and audio files. These files tend to be large, so we'd like to avoid having to download them repeatedly from our application servers or object stores (which we'll get into later).

There are a couple of types of CDNs:

A **push CDN** pre-emptively populates content that we know will be accessed in the CDN. For example, a streaming service like Netflix may have content that comes out every month that they'd anticipate their subscribers will consume, so they pre-load it onto their CDNs

A **pull CDN**, in contrast, only populates the CDN with content when a user requests it. These are useful for when we don’t know in advance what content will be popular.

### CDNs in the Wild

- [Akamai](https://www.akamai.com/solutions/content-delivery-network) provides CDN solutions among other edge computing services
- [Cloudflare](https://www.cloudflare.com/application-services/products/cdn/) provides a free CDN service
- [AWS CloudFront](https://aws.amazon.com/cloudfront/) is AWS's CDN offering

## Additional Reading / Material

- **Ilija Eftimov's Blog** ["When and Why to use an LFU cache with an implementation in Golang"](https://ieftimov.com/posts/when-why-least-frequently-used-cache-implementation-golang/)
- **jordanhasnolife System Design 2.0 Playlist**:
  - ["Introduction to Distributed Caching"](https://www.youtube.com/watch?v=crPoHnhkjFE&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=51)
  - ["Distributed Cache Writes"](https://www.youtube.com/watch?v=ULgXBImWVWQ&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=52)
  - ["Cache Eviction Policies"](https://www.youtube.com/watch?v=4wEQ9_tkqvE&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=53)
  - ["Redis vs. Memcached: Who Wins?"](https://www.youtube.com/watch?v=4wEQ9_tkqvE&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=53)
  - ["Content Delivery Networks"](https://www.youtube.com/watch?v=h5YK640kwXY&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=55)
