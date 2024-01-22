# Search Indexes

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
