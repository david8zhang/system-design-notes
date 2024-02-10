# Consistent Hashing

You might think a natural way to partition data is with the _modulo_ operator. If we have 3 partitions, for example, every key gets assigned to the partition corresponding to the hash of that key modulo 3.

But then what if one of the partitions goes down? Then we need to repartition across our whole dataset since now every key should be assigned to the partition corresponding to its hash modulo 2.

![mod-partition](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/mod-partition.png?alt=media&token=0922ba17-c6bc-4d68-bb30-1b4a76a7c33d)

Is there some way we could rebalance the keys from _only_ the partition that went down?

We can accomplish this through **consistent hashing**. In this scheme, we define hash ranges for every partition in such a way that whenever a partition goes down, we can extend the range for the remaining partitions to include the keys that were part of the partition that just went down.

In the event that the partition comes back online, we just reallocate those keys back to the partition in which they originally belonged.

![consistent-hashing](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/consistent-hashing.png?alt=media&token=00c0520c-9089-4335-8a5a-9a8e169e1ba8)

ByteByteGo also has a great [video explanation of how this process works](https://youtu.be/UF9Iqmg94tk?si=ReiA315ePHGhSKOR&t=163)
