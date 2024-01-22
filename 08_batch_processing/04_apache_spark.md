# Apache Spark

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
