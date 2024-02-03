# Read-Committed Isolation

Read committed isolation is the idea that a database query only sees data committed to _before_ the query began and never sees uncommitted data or changes committed by concurrent transactions. This ensures that our database is protected against dirty writes and dirty reads

- **Dirty writes:** When two concurrent writes conflict with each other and cause inconsistency in the database
  - Solution is row-level locking - if I’m writing to this row, it’s locked and you can’t do anything to it until I release that lock
- **Dirty read:** Reading uncommitted data: data is modified by a pending write, which causes inconsistency when reading (for example in the event that the write fails)

In practice, Read Committed isolation can be enforced as an isolation level setting for all transactions processed by your DBMS. It provides an _intermediate level_ of isolation when compared to other isolation levels:

- **Read Uncommitted (Low)**: Allows reading uncommitted data (dirty reads)
- **Read Committed (Intermediate)**: Allows only reading of committed data. However, reading a value twice may result in different values (non-repeatable)
- **Read Repeatable (High)**: Allows only reading of commited data and only by a single transaction at a time using exclusive read locks.
- **Serializable (Highest)**: Serializable execution pretty much guantees that transactions appear to be executing in serial order and, by definition, provide the highest isolation level
