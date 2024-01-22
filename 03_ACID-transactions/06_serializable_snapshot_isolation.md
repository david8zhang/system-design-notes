# Serializable Snapshot Isolation

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
