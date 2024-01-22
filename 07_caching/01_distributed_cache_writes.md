# Distributed Cache Writes

There are a few different methods for writing to a distributed cache.

## Write-Around Cache

A **write-around** caching strategy entails sending writes directly to the database, going "around" the cache. Of course, this means that the corresponding cached value will now be stale. We have a couple of ways to fix this:

1. Stale Read / TTL - We set a Time To Live (TTL) on the value in the cache, which is like an expiration date. Reads to the cache key will be stale until the TTL expires, at which point the value will be removed from the cache. A subsequent read on that key will go to the database, which will then repopulate the cache and set a new TTL

2. Invalidation - After writing to the database, we also invalidate its corresponding key in the cache. Subsequent reads will trigger the database to repopulate the cached value.

## Pros and Cons of Write-Around

- _Pros:_ Database is the source of truth, and writes are simple since they're not that different from our usual, non-cache flow
- _Cons:_ Cache misses are expensive since we need to make 2 network calls, 1 for fetching the value from the cache and 1 for fetching the value from the database.

## Write-Through Cache

A **write-through** caching strategy is when we send writes to the cache, then proxy that request to the database. This means we may have inconsistent data between our cache and our database. Sometimes that isn't an issue, but in the cases where it is, we'd need to use two phase commit (2PC) to guarantee correctness.

## Pros and Cons of Write-Through

- _Pros:_ Data is consistent between the cache and the database.
- _Cons:_ Correctness issues if we don’t use 2PC. If we do, 2PC is slow.

## Write-Back Cache

A **write-back** caching strategy is similar to the write-through caching strategy, except writes aren’t propagated to the database immediately. We do this mainly to optimize for lower write latency since we’re writing to the cache without having to also write to the database. The database write-back updates are performed asynchronously - at some point down the line, or maybe on a fixed interval, we group writes in the cache and send them to the database together.

There are a few problems that could occur - for one thing, if the cache just fails, then writes never go to the database and we have incorrect data. We can provide better fault tolerance through replication, though this may add a lot of complexity to the system.

Furthermore, if a user tries to read directly from the database before the write backs can happen, they may see a stale value. We can mitigate this with a distributed lock:

1. Whenever we write to the key in our cache, we also grab the distributed lock on that key.
2. We hold the lock until somebody tries to also grab that lock while reading that key in the database.
3. This other attempt to grab the lock would trigger a write back from the cache to the database, allowing the reader to see the up to date value.

Again, this adds a lot of complexity and latency, so we typically try to avoid needing to do this.

## Pros and Cons of Write-Back

- _Pros:_ Low latency writes.
- _Cons:_ We may have stale or incorrect data
