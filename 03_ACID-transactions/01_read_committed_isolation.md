# Read-Committed Isolation

Read committed isolation is the idea that a database query only sees data committed to _before_ the query began and never sees uncommitted data or changes committed by concurrent transactions. This ensures that our database is protected against dirty writes and dirty reads

- **Dirty writes:** When two concurrent writes conflict with each other and cause inconsistency in the database
  - Solution is row-level locking - if I’m writing to this row, it’s locked and you can’t do anything to it until I release that lock
- **Dirty read:** Reading uncommitted data: data is modified by a pending write, which causes inconsistency when reading (for example in the event that the write fails)

In practice, Read Committed isolation can be enforced as an isolation level setting for all transactions processed by your DBMS.
