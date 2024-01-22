# Batch Processing

Batch processing is the method computers use to periodically complete high volume, repetitive data jobs. Certain data processing tasks like backups, filtering, and sorting are compute intensive and inefficient to run on individual data transactions. Let's take a look at some of the systems we generally use to accomplish this.

## Hadoop Distributed File System (HDFS)

**Hadoop** is a distributed computing framework that is used for data storage (Hadoop Distributed File System) and batch processing (MapReduce or Spark).

The **Hadoop Distributed File System**, or HDFS, is a distributed, fault tolerant file store that's "rack aware". That means that we take into account the location of every computer we're storing data on in order to minimize network latency when reading and writing files.

The architecture of HDFS has two main elements: **Name Nodes**, which are used for keeping track of metadata, and **Data Nodes**, which are used for actually storing the files.

### Name Node Metadata

Every name node keeps track of metadata telling us which data node replicas store a given file, along with what version of the file the replicas have. This metadata is stored in memory (for read performance) with a write ahead log saved on disk for fault tolerance.

When a name node starts up, it asks every data node what files it contains and what versions they are. Based on this information, it replicates files to data nodes using a configurable "replication number". For example, if we setup HDFS to use a replication number of 3 and the name node sees that a file is only stored on 2 data node replicas, it will go ahead and replicate that file to a 3rd data node.

### Reading Data from HDFS

Generally we expect to read data from HDFS more often than we write it. The reading process is as follows:

1. A client asks a name node for the file location
2. The name node replies with the best data node replica for the client to read from
3. The client caches the data node replica location
4. The client reads the file from that data node replica

The "rack awareness" feature comes into play in step 2. The name node determines which replica is best for the client based on the client's proximity to it. Once the client receives that information, it can just save it in its cache rather than having to ask the name node every time.

### Writing Data to HDFS

When we write to HDFS, the name node has to select the replica it writes to in a "rack aware" manner. Here's how that plays out when we're writing a file for the first time:

1. A client tells the name node it wants to write a file
2. The name node will respond with a primary, secondary, and tertiary data node to the client based on ascending order of proximity
3. The client will write their file to the primary data node

The name node will replicate across data nodes that might be in the same data center in order to minimize network latency. For example, the primary and secondary data nodes it responds with in step 2 may be in one data center, with the tertiary data node in another.

### Replication Pipelining

Notice that the client only writes to one data node in the previous example. HDFS propagates the file to the secondary and tertiary data nodes in a process known as "replication pipelining". This process is fairly straightforward: every replica will write to the next replica in the chain, e.g. primary writes to secondary, secondary writes to tertiary.

On each successful write, an acknowledgement will be received. Under normal circumstances, the acknowledgement will propagate its way back to the client when the replication has succeeded across all data nodes. However, if there's a network failure between a primary and a secondary data node for example, the client won't receive this acknowledgement and data might not successfully replicate.

The client at this point can accept eventual consisteny, or it can continue to retry until it receives that acknowledgement. However, it's not guaranteed that that acknowledgement will ever be received. As a result, HDFS cannot be called strongly consistent.

### High Availability HDFS

Name nodes represent a single point of failure in our system if we only have one of them. HDFS solves for this issue using a coordination service like Zookeeper to keep track of backup name nodes. Coordination services are strongly consistent via the use of distributed consensus algorithms.

If a primary name node fails, its write ahead log operations stored in Zookeeper, will be replayed to the secondary name node to reconstruct the metadata information in memory. That secondary name node will then be designated as the new primary name node. This replay process is known as _state machine replication_.

### Apache HBase

**The Problem with HDFS**

One problem with HDFS is that modifying a single value in a file is very inefficient. In order to change one key, for example, we'd need to overwrite the entire file, which could be on the order of megabytes in terms of size. This is extremely expensive, especially when you factor in replication.

Apache HBase, an open-source database built on top of HDFS, can help us solve this problem. It allows for quick writes and key updates via LSM trees, as well as good batch processing functionality due to data locality from column oriented storage and range based partitioning, which keeps data with similar keys on the same partition.

**Data Model**

HBase, similar to Apache Cassandra, is a NoSQL Wide Key Store. Recall that a **wide-column database** organizes data storage into flexible columns that can be spread across multiple servers or database nodes. Each row has a required cluster and sort key, and every other column value after that is optional.

**Architecture**

Similar to HDFS, HBase maintains master nodes and partition nodes. The master node provides the same functionality as a name node in HDFS, storing metadata about data nodes and directing clients who want to read and write to their optimal data node.

The partition node, however, contains an HDFS data node as well as a _region node_. The region node stores and operates the LSM Tree in memory and flushes it to an SSTable stored in the data node when it gets to a certain capacity. The data node, as we saw in the HDFS section, will then replicate these SSTables to other data nodes in the replication pipeline.

In addition to this, HBase also uses column oriented storage, which enables it to do large analytical queries and batch processing jobs over all the values for a single column efficiently.

**When do we use HBase?**

HBase is good if you want the flexibility of normal database operations over HDFS, as well as optimized batch processing. For most applications that require write optimization, Cassandra is probably better. However there are some use cases in which HBase may be a good choice - storing application logs for diagnostic and trend analysis, or storing clickstream data for downstream analysis, for exxample.

## MapReduce

MapReduce is a programming model or pattern within the Hadoop framework that allows us to perform batch processing of big data sets. There are a few main advantages to using MapReduce:

- We can run arbitrary code, we just define custom mappers and reducers
- Run computations on the same nodes that hold the data for data locality benefits
- Failed mappers/reducers can be restarted independently

As the name implies, Mappers and Reducers are the basic building blocks of MapReduce:

- **Mappers** take an object and map it to a key-value pairing
- **Reducers** take a list of outputs produced by the mapper (over a single key) and reduce it to a single key-value pair

### Architecture

Every data node will have a bunch of unformatted data stored on disk. The MapReduce process then proceeds as follows (in memory on each data node):

- Map over all the data and turn it into key-value pairs using our Mappers
- Sort the keys. We'll explain why we do this later, but the gist is that it's easier to operate on sorted lists when we reduce.
- Shuffle the keys by hashing them and sending them to the node corresponding to the hash. This will ensure all the key-value pairs with the same key go to the same node. The sorted order of the keys is maintained on each node.
- Reduce the key-value pairs. We'll have a bunch of key-value pairs at this point that have the same key. We want to take all those and reduce it to just a single key-value pair for each key.
- Materialize the reduced data to disk

**Why do we sort our keys?**

During our reduce operation, if we have a sorted list of keys, we know that once we've seen the last key in the list, we can just flush the result to disk. Otherwise, we would have to store the intermediate result in memory in case we see another tuple for that key.

For example, assume we have the following data in our reducer and we'd like to compute the sum of values over each key:

```
Unsorted:
a: 6
a: 8
b: 3
<- At this point, we need to store the intermediate sum for "a" AND "b" in memory, since we might see more "a's" down the line
b: 7
a: 10
b: 4
```

```
Sorted:
a: 6
a: 8
a: 10
<- Once we've gotten here in our reducer, we can just flush the result for "a" to disk!
b: 3
b: 7
b: 4
```

Thus, sorting the data is much more memory efficient.

**Job Chaining**

Notice that every MapReduce job is just reading in data stored on disk for each data node, and then outputting it to data stored on disk at another or the same data node. Given that, we can actually just read in the outputs of one MapReduce as an input into another. This job chaining process allows us to achieve more complex functionality.

## Batch Joins

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

## Apache Spark

Though MapReduce is a nice framework for doing batch processing in HDFS, it isn't without its problems:

- Chained jobs in MapReduce are dependent on each other. If one takes a long time, it can block another from starting
- Each job requires a mapper and a reducer. Many times you won’t need more than one mapper, so if you have multiple jobs that all only really need 1 mapper you end up with a lot of unnecessary sorting
- Intermediate results of a MapReduce job are stored on disk, so we end up using a lot of disk space

Enter Apache Spark, which tries to address these issues:

- Nodes in Spark do computations as soon as possible instead of waiting for previous jobs to fully complete. As soon as it has all the data it needs, it proceeds to the next step
- Instead of requiring mappers and reducers, we instead have operator functions (which could be mapping or reducing)
- Spark stores intermediate states in memory instead of on disk. It only stores the input and output results on disk

Of course, the fact that Spark does everything in memory naturally raises concerns about fault-tolerance. Fortunately, it has some mechanisms to ensure that we can recover from faults:

- For narrow dependencies where computations on a given node don’t depend on data from other nodes, if a node goes down, its workload can be split across the remaining nodes and re-processed in parallel
- For wide dependencies where computations on a node depend on data from multiple other nodes, Spark writes data to disk to enable nodes to recover state after these steps are completed

## Additional Reading / Material

- _Designing Data-Intensive Applications_, Chapter 10, "Batch Processing"
- **jordanhasnolife System Design 2.0 Playlist**:
  - ["WTF is Hadoop?"](https://www.youtube.com/watch?v=ix88Zj0asjs&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=37)
  - ["What's HBase and How does it compare to Cassandra?"](https://www.youtube.com/watch?v=ouxCE6ViVpw&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=38)
  - ["WTF is MapReduce?"](https://www.youtube.com/watch?v=lHp7M078nHo&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=39)
  - ["The _Right_ Way to do Batch Job Data Joins"](https://www.youtube.com/watch?v=gqxbQTVgdkI&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=40)
