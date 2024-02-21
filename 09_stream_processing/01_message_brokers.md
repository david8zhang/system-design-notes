# Message Brokers

Instead of having many long lived connections between producers and consumers to handle event propagation, we have a _message broker_, which takes on the responsibility of ingesting events from producers and pushing them to consumers. The typical underlying implementation of these are queues.

![message-brokers](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/message-brokers.png?alt=media&token=58601c16-46e4-4101-bb40-980f720df3e9)

There are two types of message brokers: in-memory message brokers, and log-based message brokers

## In-memory Message Brokers

_Examples: RabbitMQ, ActiveMQ, Azure Service Bus, Google Cloud Pub/Sub_

![in-memory-message-broker](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/in-memory-message-broker.png?alt=media&token=3b200a00-e01c-4496-b48f-3de3ae1a4b88)

In-memory message brokers keep all messages in memory and typically optimize for higher throughput at the expense of ordered event processing and durability. Some typical use cases could include encoding user videos that get posted to Youtube, or sending user tweets to the news feeds of their followers

When a message gets delivered, it is marked for deletion from the queue. Once the consumer that received the message sends an ack that it was processed, the message is actually deleted. This could result in fault tolerance issues - if a message queue goes down, we don't have any way of replaying messages.

![in-memory-deletion](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/in-memory-broker-deletion.png?alt=media&token=ec3573e0-ca09-449c-8102-cd21125eb457)

Messages are delivered to consumers in a round robin fashion. This means that if we have multiple consumers reading messages, they may not necessarily be processed in order. If the first consumer has a slow network connection or takes longer to process the message, the second consumer may end up finishing processing the subsequent message in the queue faster.

![in-memory-out-of-order](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/in-memory-out-of-order.png?alt=media&token=47357746-6ae6-4ebd-9ac1-bc3416c38b61)

One way to avoid this is with **fan out**, where we partition our queue into multiple queues and have each consumer exclusively read from one queue. However, this would limit our throughput which kind of defeats the purpose of using an in-memory broker.

![in-memory-fanout](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/in-memory-fanout.png?alt=media&token=7e0e51ce-a396-424d-85b9-486d8cafaf68)

## Log-based Message Brokers

_Examples: Apache Kafka, Amazon Kinesis Streams_

![log-based-message-broker](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/log-based-message-broker.png?alt=media&token=56b04f13-ee10-4948-a39b-e82b15ca15d8)

A log-based message broker keeps all its messages sequentially on disk and does not delete messages after they've been consumed. That inherently gives us more durability. Some use cases for these are processing a running average of sensor metrics, or doing change data capture - keeping writes from a database in sync with another derived data store.

Every message in a log-based message broker will be stored according to the order in which it arrived. Furthermore, log based brokers keep track of which messages consumers have seen to determine what message to send next. That means every message is processed in order, so one slow to process message could potentially slow down a consumer's overall read throughput. Of course, we could mitigate this with partitioning, similar to the fan out method in which other consumers read from other queues.

![log-based-throughput](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/log-based-throughput.png?alt=media&token=88efc31f-4d91-4e95-9512-dd370a7c5216)

## Exactly once message processing

Exactly once message processing refers to the idea that we send a message both at least once _and_ no more than once

**At least once**: In order to guarantee that a message is sent at least once, our message broker will need to be fault tolerant so that it can persist messages and resend in the event of failure. It will also need consumer acknowledgement that the message was received - if there are network issues and a message is not delivered successfully (lack of acknowledgement), the broker can retry.

**No more than once**: To ensure that a message is not sent more than once, we could use Two Phase Commit to perform a distributed transaction, guaranteeing that a consumer received and processed the message and then deleting it from the broker afterwards.

![2PC-exactly-once](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/2pc-exactly-once.png?alt=media&token=5c9dbe4e-7fc5-4839-b018-0c42d2710fb0)

Otherwise, we could send the message multiple times but ensure that it isn't _processed_ more than once. In other words, messages are idempotent - sending duplicates of the same message yields the same result as sending the message only once.

![idempotence](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/idempotence-exactly-once.png?alt=media&token=f261f757-583c-477c-b828-8644755cff26)
