# GeoSpatial Indexes

Geospatial indexes specifically solve the problem of finding all points within some distance. For example, Doordash showing you the all the restaurants closest to you, or within some mile radius

Geohashes and quadtrees are the two key concepts that comprise a geospatial index.

A **quadtree** is a geographic plane split into 4 subplanes. These subplanes can then be recursively split into sub-quadtrees. A **geohash** is an address telling us which "leaf" subplane a particular point belongs to

![Quadtree](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/quadtrees.png?alt=media&token=18200f86-5894-488c-b1f3-756833011b28)

After determining the geohash of a point, we store these in sorted order (index) so that we can binary search them

```
GEOHASHES
---------
BA
BB
BC
BD
BBA
BBB
BBC
BBD
```

The neat part is that the structure of our quadtrees makes it so that points that are lexicographically close to one another are also _geographically_ close. Therefore, we can also use geohashes to partition data geographically, which is known as _geosharding_.
