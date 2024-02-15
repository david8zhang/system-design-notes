# Hadoop Distributed File System (HDFS)

**Hadoop** is a distributed computing framework that is used for data storage (Hadoop Distributed File System) and batch processing (MapReduce or Spark).

The **Hadoop Distributed File System**, or HDFS, is a distributed, fault tolerant file store that's "rack aware". That means that we take into account the location of every computer we're storing data on in order to minimize network latency when reading and writing files.

The architecture of HDFS has two main elements: **Name Nodes**, which are used for keeping track of metadata, and **Data Nodes**, which are used for actually storing the files.

![hdfs-high-level](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/hdfs-high-level.png?alt=media&token=f74d36c0-2e9d-4e57-9d85-aa424db69d3c)

## Name Node Metadata

Every name node keeps track of metadata telling us which data node replicas store a given file, along with what version of the file the replicas have. This metadata is stored in memory (for read performance) with a write ahead log saved on disk for fault tolerance.

When a name node starts up, it asks every data node what files it contains and what versions they are. Based on this information, it replicates files to data nodes using a configurable "replication number". For example, if we setup HDFS to use a replication number of 3 and the name node sees that a file is only stored on 2 data node replicas, it will go ahead and replicate that file to a 3rd data node.

![name-node-metadata](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/name-node-metadata.png?alt=media&token=2f24507a-60e7-4065-8432-e50e62e63a44)

## Reading Data from HDFS

Generally we expect to read data from HDFS more often than we write it. The reading process is as follows:

1. A client asks a name node for the file location
2. The name node replies with the best data node replica for the client to read from
3. The client caches the data node replica location
4. The client reads the file from that data node replica

![hdfs-reads](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/hdfs-reads.png?alt=media&token=8957f7e5-c81b-4fef-b4ea-81aeedb33d4f)

The "rack awareness" feature comes into play in step 2. The name node determines which replica is best for the client based on the client's proximity to it. Once the client receives that information, it can just save it in its cache rather than having to ask the name node every time.

## Writing Data to HDFS

When we write to HDFS, the name node has to select the replica it writes to in a "rack aware" manner. Here's how that plays out when we're writing a file for the first time:

1. A client tells the name node it wants to write a file
2. The name node will respond with a primary, secondary, and tertiary data node to the client based on ascending order of proximity
3. The client will write their file to the primary data node

![hdfs-writes](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/hdfs-writes.png?alt=media&token=7d27f4fc-357f-48ad-9d53-0461b2572a23)

The name node will replicate across data nodes that might be in the same data center in order to minimize network latency. For example, the primary and secondary data nodes it responds with in step 2 may be in one data center, with the tertiary data node in another.

## Replication Pipelining

Notice that the client only writes to one data node in the previous example. HDFS propagates the file to the secondary and tertiary data nodes in a process known as "replication pipelining". This process is fairly straightforward: every replica will write to the next replica in the chain, e.g. primary writes to secondary, secondary writes to tertiary. On each successful write, an acknowledgement will be received. Under normal circumstances, the acknowledgement will propagate its way back to the client when the replication has succeeded across all data nodes.

![replication-pipelining](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/hdfs-replication-pipelining.png?alt=media&token=4745ddfa-268c-443d-8d6a-8a50de1751ae)

If there's a network failure between a primary and a secondary data node for example, the client won't receive this acknowledgement and data might not successfully replicate. The client at this point can accept eventual consisteny, or it can continue to retry until it receives that acknowledgement. However, it's not guaranteed that that acknowledgement will ever be received. As a result, HDFS cannot be called strongly consistent.

## High Availability HDFS

Name nodes represent a single point of failure in our system if we only have one of them. HDFS solves for this issue by using a coordination service like Zookeeper to keep track of backup name nodes. Coordination services are strongly consistent via the use of distributed consensus algorithms.

If a primary name node fails, its write ahead log operations stored in Zookeeper, will be replayed to the secondary name node to reconstruct the metadata information in memory. That secondary name node will then be designated as the new primary name node. This replay process is known as _state machine replication_.

![high-availability-hdfs](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/high-availability-hdfs.png?alt=media&token=ee07017f-624b-4490-809e-edaa495046ec)

## Apache HBase

**The Problem with HDFS**

One problem with HDFS is that modifying a single value in a file is very inefficient. In order to change one key, for example, we'd need to overwrite the entire file, which could be on the order of megabytes in terms of size. This is extremely expensive, especially when you factor in replication.

Apache HBase, an open-source database built on top of HDFS, can help us solve this problem. It allows for quick writes and key updates via LSM trees, as well as good batch processing functionality due to data locality from column oriented storage and range based partitioning, which keeps data with similar keys on the same partition.

**Data Model**

HBase, similar to Apache Cassandra, is a NoSQL Wide Key Store. Recall that a **wide-column database** organizes data storage into flexible columns that can be spread across multiple servers or database nodes. Each row has a required cluster and sort key, and every other column value after that is optional.

![wide-column-store](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/wide-column-store.png?alt=media&token=c60e4a23-3b45-4791-a5de-37a30d6295c5)

**Architecture**

Similar to HDFS, HBase maintains master nodes and partition nodes. The master node provides the same functionality as a name node in HDFS, storing metadata about data nodes and directing clients who want to read and write to their optimal data node.

The partition node, however, contains an HDFS data node as well as a _region node_. The region node stores and operates the LSM Tree in memory and flushes it to an SSTable stored in the data node when it gets to a certain capacity. The data node, as we saw in the HDFS section, will then replicate these SSTables to other data nodes in the replication pipeline.

![apache-hbase](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/apache-hbase.png?alt=media&token=fcbcbf0a-203d-41c6-ac0d-904f38518f26)

In addition to this, HBase also uses [column oriented storage](/topic/01_storage_and_serialization?subtopic=02_column_oriented_storage), which enables it to do large analytical queries and batch processing jobs over all the values for a single column efficiently.

**When do we use HBase?**

HBase is good if you want the flexibility of normal database operations over HDFS, as well as optimized batch processing. For most applications that require write optimization, Cassandra is probably better. However there are some use cases in which HBase may be a good choice - storing application logs for diagnostic and trend analysis, or storing clickstream data for downstream analysis, for exxample.
