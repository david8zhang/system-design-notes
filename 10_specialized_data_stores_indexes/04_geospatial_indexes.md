# GeoSpatial Indexes

Geospatial indexes specifically solve the problem of finding all points within some distance. For example, Doordash showing you the all the restaurants closest to you, or within some mile radius

Geohashes and quadtrees are the two key concepts that comprise a geospatial index.

A **quadtree** is a geographic plane split into 4 subplanes. These subplanes can then be recursively split into sub-quadtrees. A **geohash** is an address telling us which "leaf" subplane a particular point belongs to

- For example, if we have a plane broken into 4 subplanes A, B, C, D, and each subplane is recursively broken into sub-subplanes A, B, C, D, a geohash for a point could be "AB", telling us the point is stored a plane A, subplane B
- After determining the geohash of a point, we store these in sorted order (index) so that we can binary search them

The neat part is that the structure of our quadtrees makes it so that points that are lexicographically close to one another are also _geographically_ close. Therefore, we can also use geohashes to partition data geographically, which is known as _geosharding_.
