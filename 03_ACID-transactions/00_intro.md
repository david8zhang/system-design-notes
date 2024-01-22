# ACID Transactions

ACID is an acronym descripting a set of properties referring to database _transactions_. Transactions are a single unit of work that access and possibly modifies the contents of a database. ACID stands for Atomicity, Consistency, Isolation, and Durability

- **Atomicity:** Every transaction either completely succeeds or completely fails (no half-measures)
- **Consistency:** The data in the database is always in a "correct" state. If there are database invariants, the data must respect those
- **Isolation:** Every concurrent database operation must produce the same result as if the operations were run in sequential order on a single thread
- **Durability:** The data can be recovered in the event of failure
