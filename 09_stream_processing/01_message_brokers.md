# Message Brokers

Instead of having many long lived connections between producers and consumers to handle event propagation, we have a _message broker_, which takes on the responsibility of ingesting events from producers and pushing them to consumers. The typical underlying implementation of these are queues.

There are two types of message brokers: in-memory message brokers, and log-based message brokers

## In-memory Message Brokers

In-memory message brokers keep all messages in memory and typically optimize for higher throughput at the expense of ordered event processing and durability. Some typical use cases could include encoding user videos that get posted to Youtube, or sending user tweets to the news feeds of their followers

When a message gets delivered, it is marked for deletion from the queue. Once the consumer that received the message sends an ack that it was processed, the message is actually deleted. This could result in fault tolerance issues - if a message queue goes down, we don't have any way of replaying messages.

Messages are delivered to consumers in a round robin fashion. This means that if we have multiple consumers reading messages, they may not necessarily be processed in order. If the first consumer has a slow network connection or takes longer to process the message, the second consumer may end up finishing processing the subsequent message in the queue faster.

One way to avoid this is with **fan out**, where we partition our queue into multiple queues and have each consumer exclusively read from one queue. However, this would limit our throughput which kind of defeats the purpose of using an in-memory broker.

## Log-based Message Brokers

A log-based message broker keeps all its messages sequentially on disk and does not delete messages after they've been consumed. That inherently gives us more durability. Some use cases for these are processing a running average of sensor metrics, or doing change data capture - keeping writes from a database in sync with another derived data store.

Every message in a log-based message broker will be stored according to the order in which it arrived. Furthermore, log based brokers keep track of which messages consumers have seen to determine what message to send next. That means every message is processed in order, so one slow to process message could potentially block all other consumers from reading. That means that in order to increase throughput, we'll need to partition our message broker.

## Exactly once message processing

Exactly once message processing refers to the idea that we send a message both at least once _and_ no more than once

- **At least once**: Requires our message broker to be fault-tolerant, requires some disk persistence and replication, and some consumer acknowledgment that the message was received
- **No more than once**: Two-phase commit using a coordinator service to provide guarantees that both the broker and consumer have sent and received the message, or idempotence in the face of receiving duplicate messages but only processing them once

## Message Brokers in the Wild:

- **In-memory**: RabbitMQ, ActiveMQ, Amazon SQS
- **Log-based**: Apache Kafka, Amazon Kinesis
