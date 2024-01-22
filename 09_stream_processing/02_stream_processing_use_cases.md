# Stream Processing Use Cases

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
