# Search Indexes

Database indexes aren’t very good for text searching because our search term might be any single substring of text stored in our DB

![indexes-for-searching](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/indexes-for-searching.png?alt=media&token=a3c2c202-ca9e-4354-bf3a-df9d52f139a0)

Search indices solve this problem using _inverted indexes_. Let's take a look at a few:

**Prefix Inverted Index**

In a prefix inverted index, we tokenize all the strings in our data and pair them with references to documents which contain those string tokens.

![prefix-inverted-index](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/prefix-inverted-index.png?alt=media&token=c9ac3095-9fb6-445e-8db0-6bff4e2c1250)

**Suffix Inverted Index**

A suffix inverted index is essentially the same as prefix-inverted indexes, except we store reversed tokens. It allows us to easily search suffixes by using the reversed the suffix as our search term string and proceeding as normal

![suffix-inverted-index](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/suffix-inverted-index.png?alt=media&token=6c2776b3-24ba-47bb-9f9c-534df20fcefa)

**Apache Lucene** is a popular open source search engine that supports complex text searches, Levenshtein distance, numerical searches, etc.

### Elasticsearch

Elasticsearch is a convenience wrapper on top of Apache Lucene to allow for searching in a distributed system. It also provides some nice features out of the box:

- REST API
- Query language
- Managed replication and partitioning
- Visualization

There are a couple of optimizations in Elasticsearch architecture that make it well suited for searching.

First, Elasticsearch caches more common/frequently accessed parts of a query instead of the full query result or a piece of an index.

Next, it uses a local secondary index as opposed to a global index. This ensures that when we're writing to our index, we won't have to potentially write to multiple different partition nodes.

![local-vs-global](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/local-vs-global-search-index.png?alt=media&token=64eead4e-654d-43ef-b78c-417dc1c117b6)

As a result, we generally want to keep searches to one partition instead of aggregating search results across many partitions
