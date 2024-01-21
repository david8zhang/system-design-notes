# Stream Processing

Unlike in batch processing, stream processing deals with _unbounded data_, or data that arrives gradually over time and is continually produced.

The general structure of stream processing systems involves two main stakeholders: producers and consumers. Producers, as the name implies, produce events. Consumers then consume those events.

## Message Brokers

Instead of having many long lived connections between producers and consumers to handle event propagation, we have a _message broker_, which takes on the responsibility of ingesting events from producers and pushing them to consumers. The typical underlying implementation of these are queues.

There are two types of message brokers: in-memory message brokers, and log-based message brokers

### In-memory Message Brokers

In-memory message brokers keep all messages in memory and typically optimize for higher throughput at the expense of ordered event processing and durability. Some typical use cases could include encoding user videos that get posted to Youtube, or sending user tweets to the news feeds of their followers

When a message gets delivered, it is marked for deletion from the queue. Once the consumer that received the message sends an ack that it was processed, the message is actually deleted. This could result in fault tolerance issues - if a message queue goes down, we don't have any way of replaying messages.

Messages are delivered to consumers in a round robin fashion. This means that if we have multiple consumers reading messages, they may not necessarily be processed in order. If the first consumer has a slow network connection or takes longer to process the message, the second consumer may end up finishing processing the subsequent message in the queue faster.

One way to avoid this is with **fan out**, where we partition our queue into multiple queues and have each consumer exclusively read from one queue. However, this would limit our throughput which kind of defeats the purpose of using an in-memory broker.

### Log-based Message Brokers

A log-based message broker keeps all its messages sequentially on disk and does not delete messages after they've been consumed. That inherently gives us more durability. Some use cases for these are processing a running average of sensor metrics, or doing change data capture - keeping writes from a database in sync with another derived data store.

Every message in a log-based message broker will be stored according to the order in which it arrived. Furthermore, log based brokers keep track of which messages consumers have seen to determine what message to send next. That means every message is processed in order, so one slow to process message could potentially block all other consumers from reading. That means that in order to increase throughput, we'll need to partition our message broker.

### Exactly once message processing

Exactly once message processing refers to the idea that we send a message both at least once _and_ no more than once

- **At least once**: Requires our message broker to be fault-tolerant, requires some disk persistence and replication, and some consumer acknowledgment that the message was received
- **No more than once**: Two-phase commit using a coordinator service to provide guarantees that both the broker and consumer have sent and received the message, or idempotence in the face of receiving duplicate messages but only processing them once

### Message Brokers in the Wild:

- **In-memory**: RabbitMQ, ActiveMQ, Amazon SQS
- **Log-based**: Apache Kafka, Amazon Kinesis

## Stream Processing Use Cases

Here are a few common stream processing use cases:

**Metric/Log Time Grouping and Bucketing**

In Metric/Log time grouping and bucketing, we want to create time interval buckets (or tumbling window) and group events according to their timestamp

Sometimes we want to aggregate over many of these intervals in our analytics (hopping window). In this case, we can use a hashmap based on our time interval (for example, one minute), and group everything that occurs within that interval together.

Other times, we just want a sliding window (get me the events that occurred in the last 5 minutes). In this case, we can use a queue to track events as they arrive and evict old events that fall outside of our window

**Change Data Capture**

The idea behind Change Data Capture (CDC), is this: whenever we write to our database, we also want to store data derived from that change somewhere else (like a search index)

In practice, this means that writes to our database will publish an event to the message broker, which then updates the derived data store and keeps it in sync. This prevents us from needing to do two-phase commit between the database and the derived data store.

**Event Sourcing**

Event sourcing is similar to Change Data Capture, except that instead of writing to a database and then propagating those changes _via_ a message broker to another data store, we write directly to the message broker itself.

The message broker can then surface events to any variety of consumers, which can then store them in a database.

An important thing to note is that events that get stored in message brokers are _database agnostic_, whereas in CDC data that's landing the message broker is _database specific_. This allows us to easily upgrade or switch out the databases that eventually store these events as long as we provide the translation logic to do so from our broker.

Event sourcing inherently assumes that the message broker is holding on to events. So that means we'd need a log-based message broker in order to facillitate this.

## Stream Enrichment

Stream enrichment is when we want to augment our stream events with more information and send them to other parts of our system. In other words, we want to _join_ data from our streams with data from other sources (which themselves could be streams or databases) to produce some kind of aggregated or combined data.

### Stream-Stream Joins

In a stream-stream join, we want to join data from two streams together. For example, let's say we have two event producers: one for "Google Search Terms" and another for "Links clicked". We want to join events from the two so that we can correlate search terms with the resulting links that were clicked by a given user.

Since events from both producers might not come at the same time through our message broker, the consumer needs to cache events in memory, and match them with other events that correspond to the same key. In the example above, a user `Alice` may search the term "tissues", and the resulting link clicked event may not come until later. So the consumer would need to hold on to the `{ "Alice": tissues" }` search term mapping until it sees the corresponding `{ "Alice": "amazon.com" }` event and join the two.

### Stream-Table Joins

In a stream table join, we now need to join events from a stream produce with data from a database. To use the previous example, let's say we had a "Demographics" table telling us the age and gender of our users. We want to join the "Google Search Terms" producer with our "Demographics" table.

A naive way to do this is to have our consumer query the database every time it receives a "Google Search Terms" event. This is inefficient since we'd need to perform a network request to the database every time.

We can do this more efficiently by storing an in-memory copy of our "Demographics" table in our consumer and using Change Data Capture to keep it in sync with the "Demographics" table in our database. In other words, we make a new write to the database, we propagate that change to our consumer using a message broker.

We can then use that in-memory table to perform the join whenever we receive an event from our "Google Search Terms" producer.

### Table-Table Joins

In a table-table join, we want to get join results as tables change. This is different from just a normal join, which is a static, one-shot operation.

To do this efficiently, we can use Change Data Capture in a similar fashion to our stream-table join example. We keep two in-memory copies of our tables in our consumer, and then perform CDC for each table to keep those in-memory tables in sync. Of course, two tables might be too big to fit in memory for one consumer. In that case, we'd need to partition our tables by the join key and perform CDC across multiple consumers.

## Additional Reading / Material
