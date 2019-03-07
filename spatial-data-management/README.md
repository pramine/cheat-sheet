# Spatial Data Management

## Cartesian coordinate system

* X-coordinate: horizontal position
* Y-coordinate: vertical position

These are typically written as an ordered pair **(X, Y)**.

![cartesian coordinate system](https://upload.wikimedia.org/wikipedia/commons/thumb/0/0e/Cartesian-coordinate-system.svg/1200px-Cartesian-coordinate-system.svg.png)

## 3D coordinate system

This system can also be used for three-dimensional geometry, where every point in Euclidean space is represented by an ordered triple of coordinates **(X, Y, Z)**.

![x, y, z](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSs45ZPcthggcMa185ZllbV65W9c30_M6jQRr6aOX2JOwpE5v0L)

## Z-Order

In coordinate geometry, X typically refers to the horizontal axis (left to right), Y to the vertical axis (up and down), and Z refers to the axis perpendicular to the other two (forward or backward). 

Z-order is an ordering of overlapping two-dimensional objects, such as windows in a stacking window manager, shapes in a vector graphics editor, or objects in a 3D application.

## Index Structure

* **Z2**: the Z2 index uses a two-dimensional Z-order curve to index *latitude* and *longitude* for point data. This index will be created if the feature type has the geometry type **Point**. 
Aim: used to efficiently answer queries of features with *point geometry with a spatial component but no temporal component*.

* **Z3**: the Z3 index uses a three-dimensional Z-order curve to index *latitude*, *longitude*, and *time* for point data. This index will be created if the feature type has the geometry type **Point** and has a time attribute. 
Aim: used to efficiently answer queries of features with *point geometry with both spatial and temporal components*.

* **XZ2**: the XZ2 index uses a two-dimensional implementation of XZ-ordering to index latitude and longitude for non-point data. XZ-ordering is an extension of Z-ordering designed for spatially extended objects (i.e. non-point geometries such as line strings or polygons). This index will be created if the feature type has a **non-Point geometry**.
Aim: used to efficiently answer queries of features with *non-point geometry with a spatial component but no temporal component*.

* **XZ3**: the XZ3 index uses a three-dimensional implementation of XZ-ordering to index latitude, longitude, and time for non-point data. This index will be created if the feature type has a non-Point geometry and has a time attribute. 
Aim: used to efficiently answer queries of features with *non-point geometry with both spatial and temporal components*.