# Write Skew & Phantom Writes

**Write Skew**

Write skew occurs when writing a value to the database with an invariant over all the data, another concurrent write of which the original write is unaware may put the database in an inconsistent state

Consider this example where we have a table of doctors with columns "name" and "status", which could be either ACTIVE or INACTIVE:

| Name         | Status   |
| ------------ | -------- |
| Dr. Toboggan | Active   |
| Dr. Oz       | Active   |
| Dr. Phil     | Inactive |

We then have an invariant (rule) that at least 1 doctor needs to be active at all times. If two transactions concurrently try to set "Dr. Oz" and "Dr. Toboggan" to "INACTIVE", they'll both read that there are 2 "ACTIVE" doctors allowing each to set its respective doctor to "INACTIVE", violating the invariant.

In other words, the end result of BOTH writes violate the invariant. This means row-level locking doesn’t work since the invariant is applied over ALL data instead of just one row. What we need is a predicate lock over ALL rows affected by the invariant.

**Phantom Writes**

A phantom write can occur when two concurrent writes try to both add the same new row. No locks can be grabbed since the row doesn’t even exist yet, resulting in duplicate rows being added to the table.

For example, imagine we have a meeting room booking application, where rooms can be booked from 11AM to 2PM for 1 hour time slots. Users add a new entry whenever they book a room for a time slot:

| Room   | Time Slot   |
| ------ | ----------- |
| Room 1 | 1PM - 2PM   |
| Room 2 | 11AM - 12PM |

Now if two people try to book Room 2 for 1PM to 2PM at the same time, for example, they could potentially _both_ add duplicate rows to the table. That would not be good.

One way to mitigate this is to pre-populate the database with all rows that could potentially exist. This approach is known as "materializing conflicts". For example:

| Room   | Time Slot   | Status   |
| ------ | ----------- | -------- |
| Room 1 | 11AM - 12PM | RESERVED |
| Room 1 | 12PM - 1PM  | RESERVED |
| Room 1 | 1PM - 2PM   | FREE     |
| Room 2 | 11AM - 12PM | FREE     |
| Room 2 | 12PM - 1PM  | RESERVED |
| Room 2 | 1PM - 2PM   | RESERVED |

Now we can use row level locking to ensure only one user is able to book the room for a given time slot!

Of course, this approach isn't feasible for all use-cases. Some storage engines like InnoDB provide ["next-key locking" to check for duplicate rows](https://dev.mysql.com/doc/refman/8.0/en/innodb-next-key-locking.html). However, if your storage engine doesn't allow this, you may have to enforce serializable isolation, preventing concurrent transactions from occurring altogether.
