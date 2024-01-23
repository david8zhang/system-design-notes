# Actual Serial Execution

Actual Serial Execution forgoes concurrent transactions entirely. Instead, we just process transactions in serialized fashion on a single core and try to optimize processing on that single core as much as possible. Some of these optimizations could include:

- Writing everything in memory instead of on disk, enabling us to use things like hash and self-balancing tree indexes. This means we can’t store as much data
- Use stored procedures - save SQL functions in the database and only accept the parameters of the function to cut down on the amount of data sent over the network
  - Stored procedures are somewhat of an antipattern in the real world nowadays since they can lead to inflexible and hard-to-maintain code

### Actual Serial Execution in the Wild

The following systems use Actual Serial Execution to maintain isolation:

- [VoltDB](https://www.youtube.com/watch?v=hD5M4a1UVz8), an in-memory database
- [Redis](https://redis.io/docs/interact/transactions/), an open source in-memory store used as a database, cache or message broker
- [Datomic](https://www.infoq.com/articles/Architecture-Datomic/), a distributed database based on the logical query language [Datalog](https://en.wikipedia.org/wiki/Datalog)
