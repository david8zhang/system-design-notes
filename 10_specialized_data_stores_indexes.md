# Specialized Data Stores and Indexes

## Search Indexes

- Database indexes aren’t very good for text searching because our search term might be any single substring of text stored in our DB
  - For example, a traditional index isn't going to help us search “Apple” in a dataset that contains “Apple”, “Cherry”, and “Green Apple” efficiently since "Cherry" comes before "Green Apple" lexicographically
  - Our index would return the ordering Apple", "Cherry", and "Green Apple" when we want "Apple" and "Green Apple" to be next to each other
- Search indices solve this problem
  - **Prefix Inverted Index**
    - Tokenize all the strings in our data and pair them with a references to documents which contain those string tokens
    - For example, "Apple" would be a token, and the ids of documents containing "Apple" and "Green Apple" would be saved as references alongside that token
  - **Suffix Inverted Index**
    - Same principle as prefix-inverted indexes, except we store reversed tokens
    - When searching for a suffix, we can just use the reversed the suffix search term string and proceed as normal
- **Apache Lucene** is a popular open source search engine that supports complex text searches, Levenshtein distance, numerical searches, etc.

### ElasticSearch

- A convenience wrapper on top of Apache Lucene to allow for searching in a distributed system. Provides some other features as well:
  - REST API
  - Query language
  - Managed replication and partitioning
  - Visualization
- Uses local indexes per partition instead of a global index to reduce data duplication
  - Generally wants to keep searches to one partition instead of aggregating search results across many partitions
- Smart caching
  - Cache more common/frequently accessed parts of a query instead of the full query result or a piece of an index

## Time Series Databases

- Optimized for reading and writing time-series data
- Examples: Timescale DB, InfluxDB, Apache Druid
- Uses column-oriented storage, better optimized for analytical queries over time
  - Hypertable and Chunks
    - Parameterizes data around time and source
    - Optimizes reads by chunking data for each source over a limited time interval
    - Optimizes writes with LSM Tree-based indexes and storage on the same node as the sensor itself
    - Optimizes deletes by designating chunks as deletable and overwriting them

## Graph Databases

- As the name implies, these are used for representing graph data, or data that can be represented with vertices/nodes connected by edges
  - For instance, a social graph where the vertices are people and the edges represent some kind of relationship (friends with, following, is followed by, etc.)

### Non-Native Implementation

- Non-native graph databases take existing database implementations (like a relational database, for example), and provide a query language on top of it for graph traversals
  - In a relational database implementation, we'd have one table for vertices and another for edges
    - Doing a graph traversal like querying all the neighbors of a given vertex would involve two separate binary searches on both the vertices and edges tables (assuming we have an index on both tables)
    - The time it takes to perform this query will grow as we add more vertices, so this is not ideal
  - In a non-relational database implementation, we could get rid of the edges table and just store the edges alongside with the vertex for better data locality
    - To be more specific, a vertex would be represented as a JSON object with an `id` and an `edges` key, which stores an array of all the vertex ids that are adjacent to this vertex
    - However, this would still require us to search the vertex table with all the vertex ids in the `edges` array
    - Storing the full vertex in the `edges` array is not viable since we'd have to store the neighboring vertices of _those_ vertices in turn, and so on, creating a nested JSON abomination

### Native Implementation

- Native graph databases, in contrast, are structured specifically to accomodate graph data
  - In Neo4J's implementation, we again have a vertex and edges table. However, this time each vertex row contains its own memory address, as well as the memory address of one of its edges
  - In turn, the edges table will have its own memory address, as well as the memory address of the vertex it points to
  - For a vertex that has multiple edges, the edge will also have the memory address of the _next_ edge
  - It's akin to a linked list structure, where a vertex node will point to an edge node, and an edge node can point to another vertex node and another edge node
  - This means we don't have to perform binary searches on the vertices table whenever we want to find adjacent nodes. We can just follow the pointers and do it in constant time
  - Caveat: Random I/O on disk isn't great, but the time complexity savings compensate for this

## GeoSpatial Indexes

- Solves the problem of finding all points within some distance
  - For example, Doordash showing you the all the restaurants closest to you, or within some mile radius
- Two key concepts - geohashes and quadtrees
  - A quadtree is a geographic plane split into 4 subplanes
    - These subplanes can then be recursively split into sub-quadtrees
  - A geohash is an address telling us which "leaf" subplane a particular point belongs to
    - For example, if we have a plane broken into 4 subplanes A, B, C, D, and each subplane is recursively broken into sub-subplanes A, B, C, D, a geohash for a point could be "AB", telling us the point is stored a plane A, subplane B
  - After determining the geohash of a point, we store these in sorted order (index) so that we can binary search them
    - The neat part is that the structure of our quadtrees makes it so that points that are lexicographically close to one another are also _geographically_ close
  - Therefore, we can also use geohashes to partition data geographically, which is known as _geosharding_.

## Object Stores

- Big Data systems like Hadoop aren't great for storing static content because we’re not planning to run any compute over it, we just want to store and retrieve it later
  - In other words, we're wasting CPU resources when we want more disk space
- An **object store** is a service offered by a cloud provider to store static content
  - Examples of these are Amazon S3 or Google Cloud Storage
  - Handles scaling and replication
  - Cheaper than a Hadoop store
  - Utilized in a Data Lake Paradigm
    - Schemaless, can store any arbitrary file types
    - Requires exporting to Hadoop for batch processing, which can be slow

## Additional Reading

**Search Indexes**

- [3.4 ElasticSearch Training - Inverted Index Explained](https://www.youtube.com/watch?v=LUjVQVl4s34&ab_channel=DataSharkAcademy)
- [Search Indexes, Jordan Has No Life System Design Interview 0 to 1](https://www.youtube.com/watch?v=ty9DQhM32mM&ab_channel=Jordanhasnolife)
