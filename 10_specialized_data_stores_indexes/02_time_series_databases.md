# Time Series Databases

A time series database is a database optimized for reading and writing time-series data. It uses column-oriented storage, which as we've seen previously, is better optimized for analytical queries over a single column.

Time series databases use hypertables to optimize reads and writes. A **hypertable** is a way to paramterize our data based on time and source. For example, we might have 3 sensors and 3 chunks of time (1-2PM, 2-3PM, 3-4PM). We can image a hypertable as being a 3x3 grid with sensors along the X-axis and time intervals along the Y-axis:

|       | Sensor 1 | Sensor2 | Sensor3 |
| ----- | -------- | ------- | ------- |
| 1-2PM |          |         |         |
| 2-3PM |          |         |         |
| 3-4PM |          |         |         |

Each cell in the table above would correspond to a chunk of data for a given sensor between a given time interval.

Breaking data up in this way allows us to cache much more efficiently, since we can pinpoint with greater accuracy the exact data we want to load into our cache. It can allow us to optimize writes by enabling us to store these chunks in the same nodes as the sensors.

Finally, it can optimize deletes as well - since these databases typically also maintain an LSM Tree and SSTable, the traditional way to delete would be to write a tombstone to our LSM Tree and propagate that to disk. That means that deleting a bunch of data would incur the same cost as writing a bunch of data. With our chunk table, we don't need to do that - the organization of the data makes it easy to expire certain time interval chunks in disk by designating them as free memory that can just be overwritten.

## Time Series Databases in the Wild

- [TimescaleDB](https://www.timescale.com/) - an open source, relational time series database
- [InfluxDB](https://www.influxdata.com/) - an open source time series database
- [Apache Druid](https://druid.apache.org/) - a column oriented, open source distributed data store primarily used for Online Analytical Processing (OLAP) use cases
