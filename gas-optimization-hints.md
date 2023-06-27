# Gas optimization hints

Try to reduce count of `[]` operations for mappings and arrays. For example:

```TVMSolidity
Struct Point {
    uint x;
    uint y;
    uint z;
}

Point[] points;
```

Here we have 3 `[]` operations:

```TVMSolidity
points[0].x = 5;
points[0].y = 10;
points[0].z = -5;
```

We can use a temp variable:

```TVMSolidity
Point p = points[0];
p.x = 5;
p.y = 10;
p.z = -5;
points[0] = p;
```