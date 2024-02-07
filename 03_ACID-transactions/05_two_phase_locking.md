# Two Phase Locking

Two Phase Locking is a concurrency control method for enforcing transaction isolation. As the name suggests, it's based on the idea that we have 2 kinds of locks over our database rows and we should apply and release them in phases.

The two distinct phases for applying and removing locks are:

1. Expanding phase: Locks are acquired and no locks are released.
2. Shrinking phase: Locks are released and no locks are acquired.

The two types of locks we use are:

- Read locks (shared lock) for whenever we’re reading data. That prevents writes from happening to the row, but still allows other reads
- Write locks (exclusive lock) for whenever we’re writing data. This prevents both reads and other writes

## Issues with Two-Phase Locking

**Deadlocks**

Deadlocks occur when two writes are dependent on each other and neither can release their locks until the other does so in turn (a circular dependency). Let's imagine the following scenario:

Two users each maintain a shopping cart:

```
Alice: [Apples, Oranges]
Bob: [Milk, Eggs]
```

Let's imagine this is some kind of "social shopping" app where each user can see each other's cart. If Alice reads from Bob's cart and decides she wants to add Bob's items to her own, she will execute a transaction with the following steps.

1. Grab a read lock on Bob's cart (to read his items)
2. Grab a read lock on Alice's cart (since we need to read in her items before we can update)
3. Grab a write lock on Alice's cart to do the update

However, what if Bob also does the same thing?

1. Grab a read lock on Alice's cart
2. Grab a read lock on Bob's cart
3. Gragb a write lock on Bob's cart to do the update

Now, we have a problem. Neither Alice nor Bob can perform step 3 and grab write locks on their own carts to update them, since write locks are exclusive. So in order for Alice to grab her write lock, she will need Bob to release his read lock on her cart. But Bob can't do that until _his_ write completes.

The only way forward would be for this to be detected by the system, and one transaction would be forced to abort.

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
