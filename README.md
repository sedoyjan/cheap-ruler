# cheap-ruler [![Build Status](https://travis-ci.org/mapbox/cheap-ruler.svg?branch=master)](https://travis-ci.org/mapbox/cheap-ruler)

A collection of fast approximations to common geographic measurements, along with some utility functions.
Useful for speeding up analysis scripts when measuring things on a city scale,
replacing [Turf](http://turfjs.org/) calls in key places.

For a city scale (a few dozen miles) and far away from poles,
the results are typically within 0.1% of corresponding Turf functions.

## Performance

Compared to corresponding Turf methods:

- `distance`: 20–25x faster
- `bearing`: 3–4x faster
- `destination`: 6–7x faster
- `lineDistance`: 20–25x faster
- `area`: 3–4x faster
- `along`: 20-25x faster
- `pointOnLine`: 70–75x faster
- `lineSlice`: 50–60x faster

Additional utility methods:

- `bufferPoint`: ~200x faster than creating a bounding box with two diagonal `turf.destination` calls
- `bufferBBox`: likewise
- `insideBBox`: ~25x faster than `turf.inside(turf.point(p), turf.bboxPolygon(bbox))`

## Usage

```js
var ruler = cheapRuler(35.05, 'miles');

var distance = ruler.distance([30.51, 50.32], [30.52, 50.312]);
var lineLength = ruler.lineDistance(line.geometry.coordinates);
var bbox = ruler.bufferPoint([30.5, 50.5], 0.01);
```

**Note**: to get the full performance benefit, create the ruler object once per an area of calculation (such as a tile), and then reuse it as much as possible.

### Creating a ruler object

#### cheapRuler(latitude[, units])

Creates a ruler object that will approximate measurements around the given latitude.
Units are either `kilometers` (default) or `miles`.

#### cheapRuler.fromTile(y, z[, units])

Creates a ruler object from tile coordinates (`y` and `z`). Convenient in `tile-reduce` scripts.

```js
var ruler = cheapRuler.fromTile(1567, 12);
```

### Ruler methods

#### distance(a, b)

Given two points of the form `[x, y]`, returns the distance.

```js
var distance = ruler.distance([30.5, 50.5], [30.51, 50.49]);
```

#### bearing(a, b)

Returns the bearing between two points in angles.

```js
var bearing = ruler.bearing([30.5, 50.5], [30.51, 50.49]);
```

#### destination(p, dist, bearing)

Returns a new point given distance and bearing from the starting point.

```js
var point = ruler.destination([30.5, 50.5], 0.1, 90);
```

#### lineDistance(line)

Given a line (an array of points), returns the total line distance.

```js
var length = ruler.lineDistance([
    [-67.031, 50.458], [-67.031, 50.534],
    [-66.929, 50.534], [-66.929, 50.458]
]);
```

#### area(polygon)

Given a polygon (an array of rings, where each ring is an array of points), returns the area.
Note that it returns the value in the specified units
(square kilometers by default) rather than square meters as in `turf.area`.

```js
var area = ruler.area([[
    [-67.031, 50.458], [-67.031, 50.534], [-66.929, 50.534],
    [-66.929, 50.458], [-67.031, 50.458]
]]);
```

#### along(line, dist)

Returns the point at a specified distance along the line.

```js
var point = ruler.along(line, 2.5);
```

#### pointOnLine(line, p)

Returns an object of the form `{point, index}` where `point` is closest point on the line from the given point,
and `index` is the start index of the segment with the closest point.

```js
var point = ruler.pointOnLine(line, [-67.04, 50.5]).point;
```

#### lineSlice(start, stop, line)

Returns a part of the given line between the start and the stop points (or their closest points on the line).

```js
ruler.pointOnLine([-67.04, 50.5], [-67.05, 50.56], line)
```

#### bufferPoint(p, buffer)

Given a point, returns a bounding box object (`[w, s, e, n]`) created from the given point buffered by a given distance.

```js
var bbox = ruler.bufferPoint([30.5, 50.5], 0.01);
```

#### bufferBBox(bbox, buffer)

Given a bounding box, returns the box buffered by a given distance.

```js
var bbox = ruler.bufferBBox([30.5, 50.5, 31, 51], 0.2);
```

#### insideBBox(p, bbox)

Returns true if the given point is inside in the given bounding box, otherwise false.

```js
var inside = ruler.insideBBox([30.5, 50.5], [30, 50, 31, 51]);
```

## Install

- NPM: `npm install cheap-ruler`
- Browser build (CDN): https://npmcdn.com/cheap-ruler@1.3.0/cheap-ruler.js
