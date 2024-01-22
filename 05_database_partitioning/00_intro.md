# Database Partitioning

Once our database starts getting too large, we'll need to split the data up across different pieces (partitions). Complications will arise in terms of how we should best split the partition, identifying partitions that a query should be routed to, and re-partitioning in the event of failure.

For choosing partitions, we have some different options:

- **Range-based partitioning:** Divide our data into ranges (of timestamp or name in alphabetical order, for example).
  - Enables very fast range queries due to data locality.
  - Could result in hot spots if a particular range is accessed more often than another.
- **Hash-range based partitioning:** Hash the key and assign it to a partition corresponding to the hash.
  - Provides a more even distribution of keys, preventing hot spots.
  - Range queries are no longer as efficient since we don’t have data locality (but we can mitigate this using indexes).
