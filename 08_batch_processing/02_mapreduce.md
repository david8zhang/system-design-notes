# MapReduce

MapReduce is a programming model or pattern within the Hadoop framework that allows us to perform batch processing of big data sets. There are a few main advantages to using MapReduce:

- We can run arbitrary code, we just define custom mappers and reducers
- Run computations on the same nodes that hold the data for data locality benefits
- Failed mappers/reducers can be restarted independently

As the name implies, Mappers and Reducers are the basic building blocks of MapReduce:

- **Mappers** take an object and map it to a key-value pairing
- **Reducers** take a list of outputs produced by the mapper (over a single key) and reduce it to a single key-value pair

## Architecture

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
