# Graph Databases

As the name implies, graph databases are used for representing graph data, or data that can be represented with vertices/nodes connected by edges

For instance, a social graph where the vertices are people and the edges represent some kind of relationship (friends with, following, is followed by, etc.)

## Non-Native Implementation

Non-native graph databases take existing database implementations (like a relational database, for example), and provide a query language on top of it for graph traversals

In a relational database implementation, we'd have one table for vertices and another for edges

- Doing a graph traversal like querying all the neighbors of a given vertex would involve two separate binary searches on both the vertices and edges tables (assuming we have an index on both tables) - The time it takes to perform this query will grow as we add more vertices, so this is not ideal

In a non-relational database implementation, we could get rid of the edges table and just store the edges alongside with the vertex for better data locality

- To be more specific, a vertex would be represented as a JSON object with an `id` and an `edges` key, which stores an array of all the vertex ids that are adjacent to this vertex
- However, this would still require us to search the vertex table with all the vertex ids in the `edges` array
- Storing the full vertex in the `edges` array is not viable since we'd have to store the neighboring vertices of _those_ vertices in turn, and so on, creating a nested JSON abomination

## Native Implementation

Native graph databases, in contrast, are structured specifically to accomodate graph data

In Neo4J's implementation, we again have a vertex and edges table. However, this time each vertex row contains its own memory address, as well as the memory address of one of its edges

In turn, the edges table will have its own memory address, as well as the memory address of the vertex it points to. For a vertex that has multiple edges, the edge will also have the memory address of the _next_ edge

It's akin to a linked list structure, where a vertex node will point to an edge node, and an edge node can point to another vertex node and another edge node

In the end, this means we don't have to perform binary searches on the vertices table whenever we want to find adjacent nodes. We can just follow the pointers and do it in constant time

**Caveat**: Random I/O on disk isn't great, but the time complexity savings compensate for this
