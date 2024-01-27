# Data Serialization Frameworks

In this section, we'll take a look at some data serialization formats and their advantages and disadvantages

## CSV

CSV stands for "Comma Separated Values". Data is this format is stored as rows of values delimited by commas. CSV is typically easy parse and read, but it doesn't guarantee type safety on the column values or even guarantee values to be present at all:

For example:

```
rownum,name,age,company
1,alice,24,albertsons
2,bob,twenty-five,amazon
3,charlie,22
```

Notice that "bob" has an string-type "age" value, and "charlie" is missing a "company" value entirely.

## JSON

JSON, or Javascript Object Notation, is a plain text, human-readable format that represents data as nested key-value pairs and arrays. JSON is widely used on the web (pretty much every language has some kind of JSON-parsing library).

Example of a JSON object

```
{
    "userName": "Mantis Toboggan",
    "age": 80,
    "interests": ["partying", "business"],
    "photoAlbum": [
        {
            "url": "images/01.jpg",
            "width": 200,
            "height": 200
        },
        {
            "url": "images/02.jpg",
            "width": 200,
            "height": 200
        }
    ]
}
```

JSON isn't the most space-efficient due to repeated keys or duplicated data (the "url", "width", and "height" tags above, for example), and also doesn't have type safety guarantees.

## XML

XML, or eXtensible Markup Language, is another plain-text, human-readable format that structures data as a series of nested tags. It's similar to HTML, except instead of using a set of predefined tags (h1, p, body), it uses custom tags defined by the author.

Example of an XML object:

```
<friendsList>
    <friend>
        <name>Mantis</name>
        <age>80</age>
    </friend>
    <friend>
        <name>Mac</name>
        <age>47</age>
    </friend>
</friendsList>
```

Like JSON, XML doesn't guarantee type saftey and isn't super memory efficient since we have repeated tags around all our data.

## Protocol Buffers and Thrift

Protocol Buffers and Thrift are binary encoding libraries developed by Google and Facebook respectively for serializing data. Here, we define a schema for data and assign each data key a numerical tag, which grants smaller data size since we just use these tags instead of strings.

Example of a Protocol Buffers schema definition:

```
message Person {
    required string user_name       = 1;
    optional int64  favorite_number = 2;
    repeated string interests       = 3;
}
```

The schemas provide type safety since they require us to specify what type everything is, as well as nice documentation for other engineers working with our system. However, writing out the schemas require manual dev effort, and the resulting encodings are also not as human-readable.

## Apache Avro

[Apache Avro](https://avro.apache.org/) is a data serialization format that was first developed for Hadoop (which you can read more about [here](/topic/08_batch_processing?subtopic=01_hdfs)). It uses JSON for defining data schemas and serializes to binary. Data in Avro is typed AND named, and columns can be compressed to save memory. It can also be read by most languages (with Java being the most widely used), though inspecting an Avro document manually requires special Avro tools since the data is serialized in binary.

Example of an Avro JSON schema:

```
{
    "type": "record",
    "name": "userInfo",
    "fields": [
        {
            "name": "username",
            "type": "string",
            "default": "NONE"
        },
        {
            "name": "favoriteNumber",
            "type": "int",
            "default": -1
        }
    ]
}
```

One nifty thing Avro can do for us is generate database schemas on the fly based off column names and reconcile different reader / writer schemas. For example, if we have two different schemas being published by two different servers writing to our database, Avro can automatically handle those by filling in missing values between the two schemas with default values.

An example scenario:

**Server A's Schema**

```
{
    "type": "record",
    "name": "userInfo",
    "fields": [
        {
            "name": "username",
            "type": "string",
            "default": "NONE"
        },
        {
            "name": "favoriteNumber",
            "type": "int",
            "default": -1
        }
    ]
}
```

**Server B's Schema**

```
{
    "type": "record",
    "name": "userInfo",
    "fields": [
        {
            "name": "username",
            "type": "string",
            "default": "NONE"
        },
        {
            "name": "age",
            "type": "int",
            "default": -1
        }
    ]
}
```

If we try to read data published by Server A using Server B's schema, Avro will automatically _ignore_ the "favoriteNumber" type. Since that data won't contain an "age" column, it will automatically use our default value of -1.
