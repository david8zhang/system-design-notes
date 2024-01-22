# Consistent Hashing

You might think a natural way to partition data is with the _modulo_ operator. If we have 3 partitions, for example, every key gets assigned to the partition corresponding to the hash of that key modulo 3.

But then what if one of the partitions goes down? Then we need to repartition across our whole dataset since now every key should be assigned to the partition corresponding to its hash modulo 2. This is obviously not ideal - we need some way to only rebalance the keys from the partition node that went down.

One way that we can accomplish just that is to use consistent hashing instead. In this scheme, we define hash ranges for every partition in such a way that whenever a partition goes down, we can extend the range for the remaining partitions to include the keys that were part of the partition that just went down.

In the event that the partition comes back online, we just reallocate those keys back to the partition in which they originally belonged.
