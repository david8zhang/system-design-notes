# Relational/Non-Relational Data

Relational data, or the relational model for representing data, is an intuitive, straightforward way of representing data in tables. Each table in our relational database only represents one type of data model. Relationships between tables are represented using _foriegn IDs_, which map two rows in two tables together. For example:

We have a companies table representing our companies

| Id  | Company   |
| --- | --------- |
| 1   | Company A |
| 2   | Company B |
| 3   | Company C |

We have an cities table representing cities

| Id  | City Name     |
| --- | ------------- |
| 1   | San Francisco |
| 2   | Seattle       |
| 3   | New York      |

A company offices table could be the result of joining the two tables, which tells us which companies have offices in which cities.

| CompanyId | CityId |
| --------- | ------ |
| 1         | 1      |
| 1         | 3      |
| 2         | 2      |
| 2         | 3      |
| 3         | 1      |

Relational data is sometimes referred to as "normalized" data. A **relational database** is a type of database that stores this data, and typically uses SQL (Structured Query Langage) for querying and updating.

## Disadvantages of Relational Data

Relational database tables in a single node might not be stored near each other on disk (poor data _locality_). That means trying to do the join across two tables could be slow due to random I/O on disk. In a distributed system, these tables might not even live on the same database node due to _partitioning_ (which we'll get into later). This would require us to make multiple network requests to different places, among other problems related to data consistency.

## Non-Relational Data

Nonrelational data uses a _denormalized_ data model. For example, we could represent the same "company offices" relation above as a dictionary:

```
{
  "Company A": ["San Francisco", "New York"],
  "Company B": ["Seattle", "New York"],
  "Company C": ["San Francisco"]
}
```

Now, we have better data locality since we don't have to query a company table and cities table separately to get the joined company offices results, everything we need is contained right there in the dictionary.

However, this means that we have repeated data keys ("San Francisco", and "New York"). Not only does this mean we need to store more data, this also means modifying our data could potentially be more complicated. If we wanted to remove "New York" from our list of cities, we'd need to update our data in multiple places.

## Relational vs. Non-Relational?

In general, we want to use non-relational data when all of our data is disjoint. For example if we have posts on Facebook, they're typically not related to each other, and can be represented in a denormalized fashion. However, if we need to represent data types that might be related, such as which authors wrote certain books, we might be better served going with a relational database.
