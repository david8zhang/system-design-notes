# ACID Transactions

ACID is an acronym descripting a set of properties referring to database _transactions_. Transactions are a single unit of work that access and possibly modifies the contents of a database. ACID stands for Atomicity, Consistency, Isolation, and Durability

- **Atomicity:** Every transaction either completely succeeds or completely fails (no half-measures)
- **Consistency:** The data in the database is always in a "correct" state. If there are database invariants, the data must respect those
- **Isolation:** Every concurrent database operation must produce the same result as if the operations were run in sequential order on a single thread
- **Durability:** The data can be recovered in the event of failure

## Read-Committed Isolation

Read committed isolation is the idea that a database query only sees data committed to _before_ the query began and never sees uncommitted data or changes committed by concurrent transactions. This ensures that our database is protected against dirty writes and dirty reads

- **Dirty writes:** When two concurrent writes conflict with each other and cause inconsistency in the database
  - Solution is row-level locking - if I’m writing to this row, it’s locked and you can’t do anything to it until I release that lock
- **Dirty read:** Reading uncommitted data: data is modified by a pending write, which causes inconsistency when reading (for example in the event that the write fails)

In practice, Read Committed isolation can be enforced as an isolation level setting for all transactions processed by your DBMS.

## Snapshot Isolation

Snapshot isolation is a guarantee that all transactions will see a consistent _snapshot_, or state, of the database and will only successfully commit if no updates it has made conflict with any concurrent updates made since that snapshot.

Snapshots similar to a write-ahead log, in that they display the last committed values in a database for a given point in time.

Every time we complete a transaction, we store the resulting value for a given key alongside a timestamp. We hold on to previous values for a given key so that at any given time we can see what the last written value was.

## Write Skew & Phantom Writes

**Write Skew**

Write skew occurs when writing a value to the database with an invariant over all the data, another concurrent write of which the original write is unaware may put the database in an inconsistent state

Consider this example where we have a table of doctors with columns "name" and "status", which could be either ACTIVE or INACTIVE:

| Name         | Status   |
| ------------ | -------- |
| Dr. Toboggan | Active   |
| Dr. Oz       | Active   |
| Dr. Phil     | Inactive |

We then have an invariant (rule) that at least 1 doctor needs to be active at all times. If two transactions concurrently try to set "Dr. Oz" and "Dr. Toboggan" to "INACTIVE", they'll both read that there are 2 "ACTIVE" doctors allowing each to set its respective doctor to "INACTIVE", violating the invariant.

In other words, the end result of BOTH writes violate the invariant. This means row-level locking doesn’t work since the variant is applied over ALL data instead of just one row. What we need is a predicate lock over ALL rows affected by the invariant.

**Phantom Writes**

When two concurrent writes try to both add the same new row. No locks can be grabbed since the row doesn’t even exist yet, resulting in duplicate rows being added to the table.

One way to mitigate this is to pre-populate the database with all rows that could potentially exist.

## Actual Serial Execution

Actual Serial Execution forgoes concurrent transactions entirely. Instead, we just process transactions in serialized fashion on a single core and try to optimize processing on that single core as much as possible. Some of these optimizations could include:

- Writing everything in memory instead of on disk, enabling us to use things like hash and self-balancing tree indexes. This means we can’t store as much data
- Use stored procedures - save SQL functions in the database and only accept the parameters of the function to cut down on the amount of data sent over the network
  - Stored procedures are somewhat of an antipattern in the real world nowadays

### Actual Serial Execution in the Wild

The following systems use Actual Serial Execution to maintain isolation:

- [VoltDB](https://www.youtube.com/watch?v=hD5M4a1UVz8), an in-memory database
- [Redis](https://redis.io/docs/interact/transactions/), an open source in-memory store used as a database, cache or message broker
- [Datomic](https://www.infoq.com/articles/Architecture-Datomic/), a distributed database based on the logical query language [Datalog](https://en.wikipedia.org/wiki/Datalog)

## Two Phase Locking

Two Phase Locking is a concurrency control method for enforcing transaction isolation. As the name suggests, it's based on the idea that we have 2 kinds of locks over our database rows and we should apply and release them in phases.

The two distinct phases for applying and removing locks are:

1. Expanding phase: Locks are acquired and no locks are released.
2. Shrinking phase: Locks are released and no locks are acquired.

The two types of locks we use are:

- Read locks (shared lock) for whenever we’re reading data. That prevents writes from happening to the row, but still allows other reads
- Write locks (exclusive lock) for whenever we’re writing data. This prevents both reads and other writes

### Issues with Two-Phase Locking

**Deadlocks**

Deadlocks occur when two writes are dependent on each other and neither can release their locks until the other does so in turn (a circular dependency). These would need to be detected by the system, and one transaction would be forced to abort

**Phantom writes**

As discussed earlier, this is when we try to acquire locks on rows don't yet exist. Let's imagine we have a table of doctor's appointments, and each doctor can only see one patient at each particular time slot

| DoctorName   | PatientName | Time    |
| ------------ | ----------- | ------- |
| Dr. Toboggan | Charlie     | 2:00 PM |
| Dr. Toboggan | Mac         | 3:00 PM |
| Dr. Oz       | Dennis      | 1:00 PM |

Let's imagine 2 patients and both try to schedule a 4:00PM appointment with Dr. Toboggan. They would each individually execute the following query:

```
SELECT * FROM Patients WHERE DoctorName = "Dr. Toboggan" AND Time="4:00PM";
```

2PL would allow both transactions to execute since they'd both be grabbing a shared read lock. Now they both grab a write lock for their write queries

```
INSERT INTO Patients ("Dr. Toboggan", "Dee", "4:00PM");
INSERT INTO Patients ("Dr. Toboggan", "Cricket", "4:00PM");
```

And since those rows don't exist yet, 2PL would allow them to do so, violiating consistency.

Similar to our write skew example, one way to mitigate this is to use a _predicate write lock_ over all rows that meet a certain predicate condition. In this case, we want to lock on predicate `DoctorName = "Dr. Toboggan" AND Time="4:00PM"`

However, these are slow to run since they require the full query to evaluate. We can do a little better if we have an index over the `DoctorName` column, which allows us to grab all of Dr. Toboggan's patients more efficiently. But then we end up write locking Dr. Toboggan's 2:00PM and 3:00PM appointments

- Thus, a predicate lock would be _pessimisstic_, since we lock more rows than we actually need.

## Serializable Snapshot Isolation

The basis of Serializable Snapshot Isolation (SSI) is: instead of pessimistically locking over many rows in the database, we can instead just save transaction values to a snapshot and then abort and rollback if we detect any inconsistencies. The general flow looks like the following:

- Whenever a write to a row begins, we log the fact that the transaction has begun in our snapshot (importantly, this log event indicates that the write has started and NOT that it has finished!)
- When we then try to read that row before the transaction completes, we read the value of the row PRIOR to the write (Read committed isolation)
- Then, upon the transaction completing, we log a commit event
- If we start any operations that depend on an uncommitted value, we'll have to check to see if we need to abort once the value is committed

Here's an example of what this might look like. Assume we have an invariant in our system saying we can only add a new appointment for a doctor if their status is ACTIVE

- **T1**: Read "Dr. Toboggan" is ACTIVE
- **T2**: Write "Dr. Toboggan" status to be INACTIVE
- **T3**: Read "Dr. Toboggan" is ACTIVE (Note that there's an uncommitted write to this value)
- **T2**: Commit
- **T3**: Add a new appointment row to the appointments table if "Dr. Toboggan"
- **T3**: Commit (and is aborted by the database)

The abort and rollback process of transactions that conflict is expensive, so SSI is best used in situations where most transactions don't conflict. This allows us to avoid unnecessarily locking rows we aren't writing to. However in other cases where transactions are overlapping like this, we should use 2PL.

### SSI in the Wild

- [FoundationDB](https://www.foundationdb.org/files/fdb-paper.pdf), a free and open source distributed NoSQL database designed by Apple, uses SSI

## Additional Reading / Material

- _Designing Data-Intensive Applications_, Chapter 7: "Transactions"
- **jordanhasnolife System Design 2.0 Playlist**:
  - ["Intro to ACID Database Transactions"](https://www.youtube.com/watch?v=oGmxzUBCYtY&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=7)
  - ["Read Committed Isolation"](https://www.youtube.com/watch?v=oS60pr8H1e0&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=8)
  - ["Snapshot Isolation"](https://www.youtube.com/watch?v=Tgpa9TrxsfU&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=9)
  - ["Write Skew and Phantom Writes"](https://www.youtube.com/watch?v=eym48yrObhY&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=10)
  - ["Achieving ACID: Serial Execution"](https://www.youtube.com/watch?v=kN_rOaNZBng&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=11)
  - ["Database Internals: Two Phase Locking"](https://www.youtube.com/watch?v=gB7qazeSD3k&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=12)
  - ["Serializable Snapshot Isolation"](https://www.youtube.com/watch?v=4TAKYRzm_dA&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=13)
- _Distributed Computing Musings_, ["Transactions: Serializable Snapshot Isolation"](https://distributed-computing-musings.com/2022/02/transactions-serializable-snapshot-isolation/)
