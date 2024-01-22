# Snapshot Isolation

Snapshot isolation is a guarantee that all transactions will see a consistent _snapshot_, or state, of the database and will only successfully commit if no updates it has made conflict with any concurrent updates made since that snapshot.

Snapshots similar to a write-ahead log, in that they display the last committed values in a database for a given point in time.

Every time we complete a transaction, we store the resulting value for a given key alongside a timestamp. We hold on to previous values for a given key so that at any given time we can see what the last written value was.
