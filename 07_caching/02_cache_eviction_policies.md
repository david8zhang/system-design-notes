# Cache Eviction Policies

Our caches have limited storage space, so we'll need to figure out how best to manage what's stored in the cache and how to get rid of things. Our aim is to minimize cache misses, so let's take a look at how we can best do that.

## First-in First-out

![fifo-eviction](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/FIFO-eviction.png?alt=media&token=60694623-ecc7-4e8a-bc9c-e76bdf6cf99b)

The first value in the cache becomes the first value to be evicted. It's relatively simple to implement since it's basically a queue. Every time we add a new value to the cache, we delete whatever the oldest value in the cache was.

The major problem with this is that we don't consider data access patterns and might evict data that's still actively being queried. For example, even if many users are querying a specific key, that key will eventually become the oldest key added to the cache as new data is added. It will subsequently be evicted despite being the most popular key.

## Least Recently Used

![lru-eviction](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/LRU-eviction.png?alt=media&token=9d2175c6-6c23-4951-a387-7c806fbcfc4a)

A better policy for evicting data is the "Least Recently Used" (LRU) policy. This is the most commonly used eviction policy in practice. In LRU, the last _accessed_ key gets evicted, and more recently used keys get to stay in the cache. This solves the problem we saw earlier with First-in First-out - popular keys will remain cached since they will keep being used.

LRU is a bit more complicated to implement - rather than just having a queue we'd need to use a hashmap with a doubly linked list. We need a hashmap to identify the node to move to the head of the list, and a doubly-linked list to be able to shift things around in O(1) time.

## Least Frequently Used

![lfu-eviction](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/LFU-eviction.png?alt=media&token=523b0cc4-ae74-4caf-9ce5-a7aee9f6c1de)

A more sophisticated alternative is "Least Frequently Used", where we keep track of the _frequencies_ that a key is being accessed and evict according to that.

Using this policy alone also has some problems - if a key is referenced repeatedly for a short period of time and then not touched for a long time afterwards, it might stay in the cache simply because it has a high frequency count from that initial spike in popularity. In addition, new items in the cache might be removed too soon since they start with a low frequency counter.

Therefore, an LFU policy is typically best for situations in which access patterns of cached objects do not change often. For example, the Google logo will always be accessed at a fairly high rate, but a hype storm around a new logo for Instagram or Reddit might temporarily increase its recency in the cache. Once the hype dies down, those new logos will be evicted as their frequency count stops growing, while Google's remains in the cache.
