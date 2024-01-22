# Leaderless Replication

Leaderless replication forgoes designating specific nodes as leader nodes entirely. In leaderless replication, we can write to any node and read from any node. This means we have high availability and fault tolerance, since every node is effectively a leader node. It also gives us high read AND write throughput.

## Quorums

To guarantee that we have the most recent values whenever we read from the system, we need a **quorum**, which just means "majority"

Writers write to a majority of nodes so that readers can guarantee that at least one of their return values will be the most recent when they read from a majority of nodes. In mathematical terms:

```
W (# of nodes we write to) + R (# of nodes we read from) > N (# of total nodes)
```

A nifty trick we can also do with quorums is **read repair**. Whenever we read values from R nodes, we might see that some of the results are out of date. We can then write the updated value back to their respective nodes.

There are some issues with quorums:

- We can still have cases in which writes arrive in different orders to a majority of nodes, causing disagreement amongst them as to which one is actually the most recent.
- Writes could also just fail, violating that inequality condition we just defined.

### Sloppy Quorums

Let's imagine a client is able to talk to _some_ database nodes during a network interruption, but not _all_ the nodes it needs to assemble a quorum. We have two options here:

1. Return errors for all requests for which we can't reach a quorum of nodes
2. Accept writes anyways, but write them to nodes that _are_ reachable, but which aren't necessarily the nodes that we normally write to.

The 2nd option causes a _sloppy quorum_ where the _W_ and _R_ in our inequality aren't among the designated _N_ "home" nodes. For example, if replication nodes for the US region fail, we could establish a quorum using some nodes from the EU region instead.

Once the original home nodes come back up, we need to propagate the writes that were sent to those temporary writer nodes back to those home nodes. This process is called _hinted handoff_.

## Anti-Entropy

Another way to prevent stale reads is to propagate writes in the background between nodes. For example, if node A has writes 1, 2, 3, 4, 5, and node B only has writes 2, 3, 5, we'd need the first node to send writes 1 and 4 over.

One way to do this is to just send the entire replication log with all the writes from node A. But this would be inefficient since all we need is the diff (just writes 1 and 4).

We can quickly obtain this diff using a **Merkle Tree**, which is a tree of hashes computed over data rows. Each individual row gets hashed to a value, and those values are combined and hashed hierarchically until we get a root hash over all the rows.

Using a binary tree search, we can efficiently identify what's changed in a dataset by comparing hash values. For example, the root hash will tell us if there is any change across our entire data set, and we can examine child hashes recursively to track down which specific rows have changed.

## Leaderless Replication in the Wild

- Amazon's [Dynamo paper](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) popularized the leaderless replication architecture. So systems that were inspired by it support leaderless replication:
  - Apache Cassandra
  - Riak
  - [Voldemort](https://www.project-voldemort.com/voldemort/), a distributed key-value store designed by LinkedIn for high scalability
- Interestingly, AWS DynamoDB does _NOT_ use leaderless replication despite having _Dynamo_ in the name
