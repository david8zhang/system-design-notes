# Functional partitioning (Federation)

Federation (or functional partitioning), splits up databases by function. For example, an e-commerce application like Amazon could break up their data into users, reviews, and products rather than having one monolothic database that stores everything.

![federation](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/federation.png?alt=media&token=a1906443-075d-4a34-b605-03625b97aa9e)

Federated databases are accompanied by a **Federated Database Management System** (FDBMS), which sits on top of the various domain-specific databases and provides a uniform interface for querying and storing data. That means users can store and retrieve data from multiple different data sources with just a single query. In addition, each of these individual databases is autonomous, managing data for its specific function or feature independently of others. Thus, an FDBMS is known as a _meta_ database management system.

Splitting up data in this manner enables greater flexibility in terms of the types of data sources that we use. Different functional domains can use different data storage technologies or formats, and our federation layer serves as a layer of abstraction on top. That also means we have greater extensibility. When an underlying database in our federated system changes, our application can still continue to function normally as long as that database is able to integrate with the federated schema.

Of course, having this federation layer adds additional complexity to our system, since we'll need to reconcile all these different data stores. Performing joins over different databases with potentially different schemas may be complex, for example. Furthermore, having to do all this aggregation logic could impact performance.
