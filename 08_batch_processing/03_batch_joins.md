# Batch Joins

During batch computations, we might want to join multiple data sets together. For example, if we have some data like the following:

```
Favorite Websites:
---------------------
alice: "google.com"
alice: "facebook.com"
bob: "instagram.com"
bob: "reddit.com"

Age / Gender:
---------------------
alice: 22, Female
bob: 23, Male
```

We might want to produce the following joined result:

```
Favorite Websites with Age/Gender
---------------------------------
alice: "google.com", 22, Female
alice: "facebook.com", 22, Female
bob: "instagram.com", 23, Male
bob: "reddit.com", 23, Male
```

We have a few ways to do this:

**Sort Merge Join**

In a **sort merge join**, we sort the keys in each partition, then hash the keys to assign them to the same partition, and then join them together, effectively merging sorted lists

This can be slow since we need to sort all the data by the join key, and we’ll need to send at least one whole dataset over the network, possibly both depending on how they are partitioned

**Broadcast Hash Join**

If we have a small dataset, we can send it to all partitions and store them entirely in memory as a hash map. That's the basis for a **Broadcast Hash Join**, in which we linearly scan through our large dataset and check it against our hash map to join it.

In this case, we don't need to sort our large dataset at all, saving a lot of time. Plus, sending _just_ the small dataset over the network can be much more efficient.

**Partition Hash Join**

But what if neither dataset is small enough to fit in memory? That's where the **partition hash join** comes in - we can just _partition_ the our datasets into smaller datasets so that they _do_ fit in memory. From there, we can just do a normal hash join.

It's important here that we partition both our datasets the same way so that the keys in each partition correspond to each other and go to the same node.
