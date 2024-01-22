# Functional partitioning (Federation)

Federation (or functional partitioning), splits up databases by function. An example of this is an e-commerce application like Amazon breaking up databases into users, reviews, and products rather than having one monolothic database that stores everything.

This gives us greater read and write throughput and minimizes replication lag. In addition, since datasets are smaller, we can cache more effectively since we can fit more of the dataset in memory. However, application logic will need to keep track of which database to read and write to and joins may end up being more complicated.
