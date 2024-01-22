# Data Serialization Frameworks

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
