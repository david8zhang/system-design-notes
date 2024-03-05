# Stream Processing Frameworks

As we saw in the previous join examples, we tend to need to keep state in memory in the consumer. As a result, we need a way to ensure fault tolerance in our consumers. Stream processing frameworks have mechanisms to ensure just that. Some examples of them include Flink, Spark Streaming, Tez, and Storm.

Note that all of these stream processing frameworks aren't the message queues, rather, they're the consumers.

## Apache Flink

Apache Flink is an open source, unified stream and batch processing framework. It allows us to have fault tolerance in our consumers and guarantees that each message only affect the state of each consumer once. An important caveat is that this guarantee only applies to consumers within the confines of our stream framework. We cannot guarantee that state outside of the framework will be affected only once when we replay messages after a crash.

**Why is Fault Tolerance Hard?**

Let's imagine a scenario where a consumer reads a message from queue A, performs some kind of stream join, and publishes the result to queue B. Before it's able to respond to queue A that it's successfully processed the message, it goes down. This means that queue A never moves to the next message in the queue, and so when the consumer comes back up, the same message will be processed and pushed to queue B again.

![stream-proc-fault-tolerance](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/fault-tolerance-stream-proc.png?alt=media&token=7dc0f205-919e-4982-888e-380ec46159f9)

**Flink Fault Tolerance**

Flink solves this issue by saving checkpoints of each consumer on some form of durable storage (such as HDFS). These checkpoints contain the state of every consumer, so in the event of failures, the consumer states can be rebuilt from those checkpoints. Furthermore, since we're using log-based queues in these kinds of scenarios, Flink can replay the messages that occurred from our checkpoint onwards.

Here's how it works: Flink designates a Job Manager node in the system that occasionally sends a barrier message through our queue, which, when received by a consumer, triggers a checkpoint to be created and saved.

![apache-flink](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/apache-flink.png?alt=media&token=08b31ff8-a8f8-49db-88fe-57cb382e3421)

These barrier messages propagate through all the consumers in our system, creating checkpoints that are _causally consistent_. Checkpoints for a consumer _only_ get created if it recieves a barrier message from all of its upstream queues its reading from. Therefore, we are guaranteed that for a given checkpoint, all of the messages that have been processed by every consumer in our system are the same.
