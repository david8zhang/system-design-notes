# B Tree / B+ Trees

B Tree indexes are database indexes that utilize a self-balancing N-ary search tree known as a B tree, and are the most widely used type of index. Unlike LSM trees and SSTables, B tree indexes are stored mostly on disk.

## B Tree Properties

![Example B-Tree](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/03_btrees.png?alt=media&token=66dacb97-1831-4add-b76a-3d0c2f205a55)

B Trees have a special property in that each node contains a range of keys in sorted order, and there is a lower and upper bound on the number of keys and children that a node may have:

- These bounds are usually determined by the size of a page on disk
- Inserting into a node might trigger a cascading series of splits to preserve the balance of the binary tree, which may incur some CPU penalties. However, the advantages we get in terms of read efficiency are typically worth these downsides

Reading from B Trees is generally fast due to the high branching factor (large number of children at each node). To be more specific, since an internal B tree node can have more than just 2 children, the logarithmic time complexity will have a higher base than 2

## B+ Trees

B+ Trees are similar to B Trees, except:

- Whereas B Trees can store data at the interior node level, B+ Trees only store data at the leaf node level
- Leaf nodes are linked together (every leaf node has a reference to the next leaf node sequentially).

The key advantage offered by this setup is that a full scan of all objects in a tree requires just one linear pass through all leaf nodes, as opposed to having to do a full tree traversal. The disadvantage is that since only leaf nodes contain data, accessing data might take longer since you have to go deeper in the tree

## B Trees / B+ Trees in the Wild

B Trees are used by most relational databases, for example:

- [InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-physical-structure.html), the storage engine of MySQL, an open source relational DBMS
- [Microsoft SQL Server](https://learn.microsoft.com/en-us/sql/relational-databases/indexes/indexes?view=sql-server-ver16), a proprietary DBMS developed and maintained by Microsoft
- [PostgreSQL](https://www.postgresql.org/docs/current/btree-implementation.html), another open source relational DBMS

However, some non-relational databases use them as well, such as

- [MongoDB](https://www.mongodb.com/docs/manual/indexes/), a document oriented NoSQL database
- [Oracle NoSQL](https://www.oracle.com/database/nosql/technologies/nosql/), a NoSQL, distributed key-value database from Oracle
