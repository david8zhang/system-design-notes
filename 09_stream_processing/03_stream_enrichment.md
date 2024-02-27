# Stream Enrichment

Stream enrichment is when we want to augment our stream events with more information and send them to other parts of our system. In other words, we want to _join_ data from our streams with data from other sources (which themselves could be streams or databases) to produce some kind of aggregated or combined data.

## Stream-Stream Joins

In a stream-stream join, we want to join data from two streams together. For example, let's say we have two event producers: one for "Google Search Terms" and another for "Links clicked". We want to join events from the two so that we can correlate search terms with the resulting links that were clicked by a given user.

![image](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/stream-stream-join.png?alt=media&token=978b728a-1056-4fb6-92f7-237a3b76f51b)

Since events from both producers might not come at the same time through our message broker, the consumer needs to cache events in memory, and match them with other events that correspond to the same key. In the example above, a user `Alice` may search the term "tissues", and the resulting link clicked event may not come until later. So the consumer would need to hold on to the `{ "Alice": tissues" }` search term mapping until it sees the corresponding `{ "Alice": "amazon.com" }` event and join the two.

## Stream-Table Joins

In a stream table join, we now need to join events from a stream produce with data from a database. To use the previous example, let's say we had a "Demographics" table telling us the age and gender of our users. We want to join the "Google Search Terms" producer with our "Demographics" table.

A naive way to do this is to have our consumer query the database every time it receives a "Google Search Terms" event. This is inefficient since we'd need to perform a network request to the database every time.

We can do this more efficiently by storing an in-memory copy of our "Demographics" table in our consumer and using Change Data Capture to keep it in sync with the "Demographics" table in our database. In other words, we make a new write to the database, we propagate that change to our consumer using a message broker.

![stream-table](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/stream-table-join.png?alt=media&token=6ae8f1ef-e1cc-4da7-8b99-fe35ae26ad6b)

We can then use that in-memory table to perform the join whenever we receive an event from our "Google Search Terms" producer.

## Table-Table Joins

In a table-table join, we want to get join results as tables change. This is different from just a normal join, which is a static, one-shot operation.

![table-table](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/table-table-join.png?alt=media&token=1e3008f5-cea1-4eb9-97b6-86f2b3e218ed)

To do this efficiently, we can use Change Data Capture in a similar fashion to our stream-table join example. We keep two in-memory copies of our tables in our consumer, and then perform CDC for each table to keep those in-memory tables in sync. Of course, two tables might be too big to fit in memory for one consumer. In that case, we'd need to partition our tables by the join key and perform CDC across multiple consumers.
