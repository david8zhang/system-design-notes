# Storage & Serialization

## Column Oriented Storage

_Row-oriented storage_ has data for a single row stored together, which is basically like your regular relational database table. For example, the Employees table stored as **employees.txt**:

| Name    | Email             | Company  |
| ------- | ----------------- | -------- |
| Alice   | alice@email.com   | Google   |
| Bob     | bob@email.com     | Amazon   |
| Charlie | charlie@email.com | Facebook |

_Column oriented storage_ stores a bunch of column values together. So rather than having the Name, Email, and Company all in the same file, we split out each column into its own file and just store the value of that column for each row. For example we'd have a **companies.txt** (and so on for the other columns):

| Company  |
| -------- |
| Google   |
| Amazon   |
| Facebook |
| ...      |

Using column oriented storage gives us several advantages:

- We can do faster analytical queries over all or a large set of the values in our data for just a single column
- Column compression can also be performed to minimize the amount of data being stored (see below)
- Since we have less data in this scenario, we can even store it in memory or CPU cache for even faster reads/writes.

### Column Compression

Imagine we have the following table:

| Name    | Followers |
| ------- | --------- |
| Alice   | 3         |
| Bob     | 3         |
| Charlie | 1         |
| David   | 1         |
| Edward  | 2         |
| Frank   | 5         |
| Gordon  | 4         |

If we have column oriented storage, we can easily compress the "Followers" file using **bitmap encoding**:

| Followers | Bitmap Encoding |
| --------- | --------------- |
| 1         | 0011000         |
| 2         | 0000100         |
| 3         | 1100000         |
| 4         | 0000001         |
| 5         | 0000010         |

The way this works is: We see that Charlie and David (3rd and 4th in our table) both have only 1 Follower, so for the 1 Follower row, we set the 3rd and 4th bits in our bitmap going left to right and leave the rest as zeroes. (Hence, 0011000 in the "1 Follower" row)

We can compress this bitmap further by using a **run length encoding**:

| Followers | Bitmap Encoding | Run-Length Encoding |
| --------- | --------------- | ------------------- |
| 1         | 0011000         | 223                 |
| 2         | 0000100         | 412                 |
| 3         | 1100000         | 25                  |
| 4         | 0000001         | 61                  |
| 5         | 0000010         | 511                 |

The run length stores the bitmap encoding as a sequence of the count of consecutive zeroes and count of consecutive ones. For example, "0011000" is 2 consecutive 0's, 2 consecutive 1's and 3 consecutive 0's, resulting in "223".

Performing column compression enables us to:

- Send less data over the network
- Potentially keep more data stored in memory or CPU cache if the dataset is small enough

### Downisides to column oriented storage

There are a few downsides to column oriented storage:

- Every column must have the same sort order.
- Writes for a single row need to go to different places on disk (This can be improved via an in-memory LSM tree/SSTable set-up since that preserves the sort order of columns).

### Column Oriented Storage in the Wild

- [Apache Parquet](https://parquet.apache.org/) is an open-source column-oriented data file format designed for efficient storage and retrieval. It provides some nice features like:
  - Metadata containing minimum, maximum, sum, or average values for each data file chunk, which enables us to efficiently perform queries
  - Efficient data compression and encoding schemes
- [Amazon Redshift](https://aws.amazon.com/redshift/) is a column-oriented managed data warehouse solution from AWS
- [Apache Druid](https://druid.apache.org/docs//0.21.0/design/index.html) is a column-oriented, open source, real-time analytics database

## Data Serialization Frameworks

When we're working with data in memory, we use data structures like lists, arrays, and hashtables, which are optimized for efficient access and manipulation by the CPU using pointers. However, when we send data over the network, we need some kind of encoding format since CPU pointers won't make sense across processes or systems.

**Relational data (SQL)**

Relational data is the classic tabular format for data, which, as the name implies, is nice for representing relations between data types. Some drawbacks include lack of data locality if we’re performing joins across tables

**JSON, XML Data**

JSON and XML are plain text, human-readable formats that are good for data locality due to denormalization. They aren't necessarily the most space-efficient due to repeated string keys/duplicated data and they also don't have type safety guarantees

Example of a JSON object

```
{
    "userName": "Mantis Toboggan",
    "age": 80,
    "interests": ["feasting on scraps", "making viral videos"]
}

```

**Protocol Buffers and Thrift**

Protocol Buffers and Thrift are binary encoding libraries developed by Google and Facebook respectively for serializing data. Here, we define a schema for data and assign each data key a numerical tag, which grants smaller data size since we just use these tags instead of strings.

The schemas also provide type safety since they require us to specify what type everything is, as well as nice documentation for other engineers working with our system. However, writing out the schemas require manual dev effort, and the resulting encodings are also not as human-readable.

Example of a Protocol Buffers schema definition:

```
message Person {
    required string user_name       = 1;
    optional int64  favorite_number = 2;
    repeated string interests       = 3;
}
```

## Additional Reading / Material

- _Designing Data-Intensive Applications_, Chapter 3, "Storage and Retrieval", section 3: "Column-Oriented Storage"
- _Designing Data-Intensive Applications_, Chapter 4, "Encoding and Evolution", section 1: "Formats for Encoding Data"
- **jordanhasnolife System Design 2.0 Playlist**:
  - ["Column Oriented Storage"](https://www.youtube.com/watch?v=Zt7rqtJ3uWA&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=14)
  - ["Data Serialization Frameworks"](https://www.youtube.com/watch?v=E7Gk8etqkgU&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=15)
