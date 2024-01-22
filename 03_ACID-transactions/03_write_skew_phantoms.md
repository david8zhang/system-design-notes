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

In other words, the end result of BOTH writes violate the invariant. This means row-level locking doesn’t work since the variant is applied over ALL data instead of just one row. What we need is a predicate lock over ALL rows affected by the invariant.

**Phantom Writes**

When two concurrent writes try to both add the same new row. No locks can be grabbed since the row doesn’t even exist yet, resulting in duplicate rows being added to the table.

One way to mitigate this is to pre-populate the database with all rows that could potentially exist.
