# Specialized Data Stores and Indexes

## Search Indexes

Database indexes aren’t very good for text searching because our search term might be any single substring of text stored in our DB

For example, a traditional index isn't going to help us search “Apple” in a dataset that contains “Apple”, “Cherry”, and “Green Apple” efficiently since "Cherry" comes before "Green Apple" lexicographically. Our index would return the ordering "Apple", "Cherry", and "Green Apple" when we want "Apple" and "Green Apple" to be next to each other

Search indices solve this problem using _inverted indexes_. Let's take a look at a few:

**Prefix Inverted Index**

In a prefix inverted index, we tokenize all the strings in our data and pair them with references to documents which contain those string tokens. To use our example from before, "Apple" would be a token, and the ids of documents containing "Apple" and "Green Apple" would be saved as references alongside that token.

**Suffix Inverted Index**

A suffix inverted index is essentially the same as prefix-inverted indexes, except we store reversed tokens. It allows us to easily search suffixes by using the reversed the suffix as our search term string and proceeding as normal

**Apache Lucene** is a popular open source search engine that supports complex text searches, Levenshtein distance, numerical searches, etc.

### ElasticSearch

ElasticSearch is a convenience wrapper on top of Apache Lucene to allow for searching in a distributed system. It also provides some nice features out of the box:

- REST API
- Query language
- Managed replication and partitioning
- Visualization
- Uses local indexes per partition instead of a global index to reduce data duplication
  - Generally wants to keep searches to one partition instead of aggregating search results across many partitions
- Smart caching
  - Cache more common/frequently accessed parts of a query instead of the full query result or a piece of an index

## Time Series Databases

A time series database is a database optimized for reading and writing time-series data. It uses column-oriented storage, which as we've seen previously, is better optimized for analytical queries over a single column.

Time series databases use hypertables to optimize reads and writes. A **hypertable** is a way to paramterize our data based on time and source. For example, we might have 3 sensors and 3 chunks of time (1-2PM, 2-3PM, 3-4PM). We can image a hypertable as being a 3x3 grid with sensors along the X-axis and time intervals along the Y-axis:

|       | Sensor 1 | Sensor2 | Sensor3 |
| ----- | -------- | ------- | ------- |
| 1-2PM |          |         |
| 2-3PM |          |         |
| 3-4PM |          |         |

Each cell in the table above would correspond to a chunk of data for a given sensor between a given time interval.

Breaking data up in this way allows us to cache much more efficiently, since we can pinpoint with greater accuracy the exact data we want to load into our cache. It can allow us to optimize writes by enabling us to store these chunks in the same nodes as the sensors.

Finally, it can optimize deletes as well - since these databases typically also maintain an LSM Tree and SSTable, the traditional way to delete would be to write a tombstone to our LSM Tree and propagate that to disk. That means that deleting a bunch of data would incur the same cost as writing a bunch of data. With our chunk table, we don't need to do that - the organization of the data makes it easy to expire certain time interval chunks in disk by designating them as free memory that can just be overwritten.

### Time Series Databases in the Wild

- [TimescaleDB](https://www.timescale.com/) - an open source, relational time series database
- [InfluxDB](https://www.influxdata.com/) - an open source time series database
- [Apache Druid](https://druid.apache.org/) - a column oriented, open source distributed data store primarily used for Online Analytical Processing (OLAP) use cases

## Graph Databases

As the name implies, graph databases are used for representing graph data, or data that can be represented with vertices/nodes connected by edges

For instance, a social graph where the vertices are people and the edges represent some kind of relationship (friends with, following, is followed by, etc.)

### Non-Native Implementation

Non-native graph databases take existing database implementations (like a relational database, for example), and provide a query language on top of it for graph traversals

In a relational database implementation, we'd have one table for vertices and another for edges

- Doing a graph traversal like querying all the neighbors of a given vertex would involve two separate binary searches on both the vertices and edges tables (assuming we have an index on both tables) - The time it takes to perform this query will grow as we add more vertices, so this is not ideal

In a non-relational database implementation, we could get rid of the edges table and just store the edges alongside with the vertex for better data locality

- To be more specific, a vertex would be represented as a JSON object with an `id` and an `edges` key, which stores an array of all the vertex ids that are adjacent to this vertex
- However, this would still require us to search the vertex table with all the vertex ids in the `edges` array
- Storing the full vertex in the `edges` array is not viable since we'd have to store the neighboring vertices of _those_ vertices in turn, and so on, creating a nested JSON abomination

### Native Implementation

Native graph databases, in contrast, are structured specifically to accomodate graph data

In Neo4J's implementation, we again have a vertex and edges table. However, this time each vertex row contains its own memory address, as well as the memory address of one of its edges

In turn, the edges table will have its own memory address, as well as the memory address of the vertex it points to. For a vertex that has multiple edges, the edge will also have the memory address of the _next_ edge

It's akin to a linked list structure, where a vertex node will point to an edge node, and an edge node can point to another vertex node and another edge node

In the end, this means we don't have to perform binary searches on the vertices table whenever we want to find adjacent nodes. We can just follow the pointers and do it in constant time

**Caveat**: Random I/O on disk isn't great, but the time complexity savings compensate for this

## GeoSpatial Indexes

Geospatial indexes specifically solve the problem of finding all points within some distance. For example, Doordash showing you the all the restaurants closest to you, or within some mile radius

Geohashes and quadtrees are the two key concepts that comprise a geospatial index.

A **quadtree** is a geographic plane split into 4 subplanes. These subplanes can then be recursively split into sub-quadtrees. A **geohash** is an address telling us which "leaf" subplane a particular point belongs to

- For example, if we have a plane broken into 4 subplanes A, B, C, D, and each subplane is recursively broken into sub-subplanes A, B, C, D, a geohash for a point could be "AB", telling us the point is stored a plane A, subplane B
- After determining the geohash of a point, we store these in sorted order (index) so that we can binary search them

The neat part is that the structure of our quadtrees makes it so that points that are lexicographically close to one another are also _geographically_ close. Therefore, we can also use geohashes to partition data geographically, which is known as _geosharding_.

## Object Stores

Big Data systems like Hadoop aren't great for storing static content because we’re not planning to run any compute over it, we generally just want to store and retrieve it later. In other words, we're wasting CPU resources when we want more disk space

An **object store** is a service offered by a cloud provider to store static content. They typically are managed services that handle scaling and replication and are schemaless; they're able to store any arbitrary file types

Data lakes, which are centralized repositories designed to store, process, and secure large amounts of structured, semi-structured, and unstructured data, are built on top of object stores since they typically stores data in its native format (blobs and files). Performing batch processing over this data would require us to export it to Hadoop for batch processing, which can be slow.

## Additional Reading / Material

- [3.4 ElasticSearch Training - Inverted Index Explained](https://www.youtube.com/watch?v=LUjVQVl4s34&ab_channel=DataSharkAcademy)
- **jordanhasnolife System Design 2.0 Playlist**:
  - ["Search Indexes"](https://www.youtube.com/watch?v=ty9DQhM32mM&ab_channel=Jordanhasnolife)
  - ["How are Time Series Databases SO FAST?"](https://www.youtube.com/watch?v=fUpYLwzGtW0&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=48&t=206s&pp=iAQB)
  - ["How are Graph Databases So Fast?"](https://www.youtube.com/watch?v=Sdw_D-Gllac&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=49&t=109s&pp=iAQB)
  - ["GeoSpatial Indexes - Why You Need Them"](https://www.youtube.com/watch?v=9BewOp5Gaw8&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=50)
