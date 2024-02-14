# Column Oriented Storage

_Row-oriented storage_ has data for a single row stored together, which is basically like your regular relational database table. For example, the Employees table stored as **employees.txt**:

| Name    | Email             | Company  |
| ------- | ----------------- | -------- |
| Alice   | alice@email.com   | Google   |
| Bob     | bob@email.com     | Amazon   |
| Charlie | charlie@email.com | Facebook |

_Column oriented storage_ stores a bunch of column values together. So rather than having the Name, Email, and Company all in the same file, we split out each column into its own file and just store the value of that column for each row. For example we'd have a **companies.txt**, **emails.txt**, and **names.txt**:

| Company  |
| -------- |
| Google   |
| Amazon   |
| Facebook |
| ...      |

(You can extrapolate the above to apply to emails and names as well)

Using column oriented storage gives us several advantages:

- We can do faster analytical queries over all or a large set of the values in our data for just a single column
- Column compression can also be performed to minimize the amount of data being stored (see below)
- Since we have less data in this scenario, we can even store it in memory or CPU cache for even faster reads/writes.

## Column Compression

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

## Downsides to column oriented storage

There are a few downsides to column oriented storage:

- Every column must have the same sort order.
- Writes for a single row need to go to different places on disk (This can be improved via an in-memory LSM tree/SSTable set-up since that preserves the sort order of columns).

## Column Oriented Storage in the Wild

- [Apache Parquet](https://parquet.apache.org/) is an open-source column-oriented data file format designed for efficient storage and retrieval. It provides some nice features like:
  - Metadata containing minimum, maximum, sum, or average values for each data file chunk, which enables us to efficiently perform queries
  - Efficient data compression and encoding schemes
- [Amazon Redshift](https://aws.amazon.com/redshift/) is a column-oriented managed data warehouse solution from AWS
- [Apache Druid](https://druid.apache.org/docs//0.21.0/design/index.html) is a column-oriented, open source, real-time analytics database
