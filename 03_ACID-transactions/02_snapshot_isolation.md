# Snapshot Isolation

## Read Skew

Consider the following example. We have the following list doctors and patients:

```
Dr. Toboggan: 2
Dr. Phil: 3
Dr. Oz: 2
Dr. Doom: 1
Dr. Patel: 2
```

Let's assume we have a query that just goes down the list one at a time. We have an invariant over the table that there are a total of 10 patients at ALL times.

However, let's now assume that before we begin reading Dr. Oz, one patient transfers from Dr. Oz to Dr. Toboggan.

So now, we have the following:

```
Dr. Toboggan: 2 (now updated to 3)
Dr. Phil: 3
Dr. Oz: 1 < READ
Dr. Doom: 1
Dr. Patel: 2
```

But now there's a problem. We've _already_ read that Dr. Toboggan only had 2 patients. However, due to this write happening in the middle of our read, we now see that a patient has gone missing. This state, in which our read is over an inconsistent state of the database, is called **read skew**.

## Snapshots

Snapshot isolation is a guarantee that all transactions will see a consistent _snapshot_, or state, of the database. This addresses the example that we saw above.

Snapshot isolation also guarantees that a write will only successfully commit if it does not conflict with any concurrent updates made since that snapshot.

Snapshots similar to a write-ahead log, in that they display the last committed values in a database for a given point in time. So in our example above, our read query (let's assume it occurs at time T1) would see a snapshot of the database prior to the write, when the patient was transferered.

```
[T1] Dr. Toboggan: 2
[T1] Dr. Phil: 3
[T1] Dr. Oz: 2
[T1] Dr. Doom: 1
[T1] Dr. Patel: 2
```

Every time we complete a transaction, we store the resulting value for a given key alongside a timestamp. We hold on to previous values for a given key so that at any given time we can see what the last written value was.
