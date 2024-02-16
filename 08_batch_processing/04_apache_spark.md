# Apache Spark

Though MapReduce is a nice framework for doing batch processing in HDFS, it isn't without its problems:

- Chained jobs in MapReduce are dependent on each other. If one takes a long time, it can block another from starting
- Each job requires a mapper and a reducer. Many times you won’t need more than one mapper, so if you have multiple jobs that all only really need 1 mapper you end up with a lot of unnecessary sorting
- Intermediate results of a MapReduce job are stored on disk, so we end up using a lot of disk space

Enter Apache Spark, which tries to address these issues:

- Nodes in Spark do computations as soon as possible instead of waiting for previous jobs to fully complete. As soon as it has all the data it needs, it proceeds to the next step
- Instead of requiring mappers and reducers, we instead have operator functions (which could be mapping or reducing)
- Spark stores intermediate states in memory instead of on disk. It only stores the input and output results on disk

## Spark Architecture

### Resilient Distributed Datasets

_Resilient Distributed Datasets_ (RDD)s are a core abstraction in Spark. They are immutable, distributed collections of elements of your data that can be operated on in parallel.

RDDs are abstractions of data collections sourced from multiple partitions and/or data stores (e.g. SQL tables, HDFS files, text files). Furthermore, Spark processes RDDs entirely in memory, which provides some nice performance benefits.

![spark-rdd](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/spark-rdd.png?alt=media&token=59fd8a6f-cfed-46e6-955e-c1abdae5af96)

### How Spark Jobs Run

Spark applications are coordinated by a main program (known as a driver), which spins up a SparkContext object. The SparkContext object orchestrates tasks by talking to a cluster manager service like [Kubernetes](/topic/13_software_architecture?subtopic=02_containers), Mesos, or Spark's own standalone cluster manager. It requests "executors" on nodes in the cluster, which are processes that perform computations and store data. The SparkContext then forwards application code (JAR or Python files) to the executors. Finally, it sends tasks to the cluster manager, which will then schedule and run them on the nodes using a Direct Acyclic Graph (DAG) Scheduler.

![spark-execution](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/spark-execution.png?alt=media&token=aa10797a-5921-4418-bc01-32408ead9aa7)

### Fault Tolerance

Of course, the fact that Spark does everything in memory naturally raises concerns about fault-tolerance. Fortunately, it has some mechanisms to ensure that we can recover from faults:

For **narrow dependencies** where computations on a given node don’t depend on data from other nodes, if a node goes down, its workload can be split across the remaining nodes and re-processed in parallel

![narrow-dependency](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/spark-narrow-dependencies.png?alt=media&token=a31fd10a-e87d-4c39-8d6b-2f598e47fd11)

For **wide dependencies** where computations on a node depend on data from multiple other nodes, the process is much more tedious. If a node fails, its upstream states will need to be recomputed. Luckily, Spark automatically checkpoints data to disk after a wide dependency.

![wide-dependency](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/spark-wide-dependency.png?alt=media&token=c6b42c7b-9285-4a8c-9675-8feceb90b65f)
