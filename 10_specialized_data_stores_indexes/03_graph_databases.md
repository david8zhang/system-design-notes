# Graph Databases

As the name implies, graph databases are used for representing graph data, or data that can be represented with vertices/nodes connected by edges

For instance, a social graph where the vertices are people and the edges represent some kind of relationship (friends with, following, is followed by, etc.)

## Non-Native Implementation

Non-native graph databases take existing database implementations (like a relational database, for example), and provide a query language on top of it for graph traversals

In a relational database implementation, we'd have one table for vertices and another for edges

**Vertices**

| id  | data    |
| --- | ------- |
| 1   | Alice   |
| 2   | Bob     |
| 3   | Charlie |
| 4   | David   |
| 5   | Eve     |

**Edges**

| From | To  |
| ---- | --- |
| 1    | 2   |
| 2    | 4   |
| 3    | 2   |
| 4    | 1   |
| 5    | 3   |

Doing a graph traversal like querying all the neighbors of a given vertex would involve two separate binary searches on both the vertices and edges tables (assuming we have an index on both tables) - The time it takes to perform this query will grow as we add more vertices, so this is not ideal

In a non-relational database implementation, we could get rid of the edges table and just store the edges alongside with the vertex for better data locality.

```json
{
  "id": 1,
  "edges": [2, 3]
}
```

However, this would still require us to search the vertex table with all the vertex ids in the `edges` array

Storing the full vertex in the `edges` array is not viable since we'd have to store the neighboring vertices of _those_ vertices in turn, and so on, creating a nested JSON abomination

```json
{
  "id": 1,
  "edges": [
    {
      "id": 2,
      "edges": [
        {
          "id": 3,
          "edges": [] // we can go deeper and deeper...
        }
      ]
    }
  ]
}
```

## Native Implementation

Native graph databases, in contrast, are structured specifically to accomodate graph data

In Neo4J's implementation, we again have a vertex and edges table. However, this time each vertex row contains its own memory address, as well as the memory address of one of its edges

**Nodes**

| Address | Name    | Edge Address |
| ------- | ------- | ------------ |
| 0x01    | Alice   | null         |
| 0x02    | Bob     | 0x07         |
| 0x03    | Charlie | 0x08         |
| 0x04    | David   | 0x09         |
| 0x05    | Eve     | 0x10         |

**Edges**

| Address | Points To | Next Edge Address |
| ------- | --------- | ----------------- |
| 0x07    | 0x01      | null              |
| 0x08    | 0x01      | null              |
| 0x09    | 0x02      | null              |
| 0x10    | 0x02      | 0x11              |
| 0x11    | 0x03      | null              |

In turn, the edges table will have its own memory address, as well as the memory address of the vertex it points to. For a vertex that has multiple edges, the edge will also have the memory address of the _next_ edge

It's akin to a linked list structure, where a vertex node will point to an edge node, and an edge node can point to another vertex node and another edge node

In the end, this means we don't have to perform binary searches on the vertices table whenever we want to find adjacent nodes. We can just follow the pointers and do it in constant time

**Caveat**: Random I/O on disk isn't great, but the time complexity savings compensate for this
