---
layout: single
title:  "Barycentric Coordinates"
date:   2014-10-20 21:45:45 -0600
categories: posts
use_math: true
header:
  overlay_image: /assets/images/barycentric/barycentric-colors-blackfill.png
  og_image: /assets/images/barycentric/barycentric-colors-blackfill.png
  teaser: /assets/images/barycentric/barycentric-colors-blackfill.png
tags:
  - barycentric coordinates
  - math
  - CFD meshing
---
Triangles and tetrahedra are commonly used elements in CFD and finite element simulations.
Barycentric coordinates enable mesh modification, interpolation, and locating the element that contains a point.
This article aims to demystify barycentric coordinates so you can use them in your own applications.

## Fundamentals

For simplicity, I will start with the two-dimensional case of a triangle, but this can be extended easily into three dimensions.  
For a triangle, the barycentric coordinate system is comprised of three dependent basis vectors. 
Each basis vector extends from the midpoint of one side of the triangle to the opposite corner as shown below.

![Barycentric coordinates.](/assets/images/barycentric/barycentric.png){: .align-center}

Notice that each corner of the triangle is now represented by a coordinate triple. 
In fact, any point $(x,y)$ can be represented by a coordinate triple $(\xi_1, \xi_2, \xi_3)$.
We also notice that any point outside the triangle will have at least one negative coordinate.
This makes barycentric coordinates ideal for determining whether a point lies inside a triangle. In addition, the coordinates of a point must always add up to 1.

$$
\xi_1 + \xi_2 + \xi_3 = 1
$$

### Barycentric Coordinates in Two Dimensions

Barycentric coordinates of a point are simple to calculate. If we take a point inside the triangle and connect that point to each vertex of the triangle, the result is three subtriangles with areas $A_1$, $A_2$, and $A_3$ as shown below. If the total area of the triangle is $A$, then the barycentric coordinates of point $P$ are $(\frac{A_1}{A}, \frac{A_2}{A}, \frac{A_3}{A})$.

![Subdivided triangle with point of interest, P, at the common vertex.](/assets/images/barycentric/barycentric_subtriangles.png){: .align-center}

The barycentric coordinates are simply a ratio of the area of each subtriangle to the total area of the triangle. 
Now you can see why the coordinates must add up to one. 
The overall area of the triangle is the sum of the areas of the three subtriangles. 
This also works for points located outside the triangle.
Area is a vector quantity, which means there is a direction associated with it. 
This means that a point outside the triangle will give a negative area for one or more triangles. 
Let's take a look at this in more detail. 

I will use the notation $\vec{\mathbf{V}}_{ab}$ to represent a vector from point $a$ to point $b$ in the figure above, and similarly for other vectors. The normal vector of the triangle and its subtriangles are then given by the cross product of the vectors that define 2 of their sides, as follows:

$$
\begin{aligned} 
\vec{\mathbf{n}}&=\vec{\mathbf{V}}\_{ab} \times \vec{\mathbf{V}}\_{ac} \\
\vec{\mathbf{n}}\_1&=\vec{\mathbf{V}}_{bc} \times \vec{\mathbf{V}}\_{bp} \\
\vec{\mathbf{n}}\_2&=\vec{\mathbf{V}}_{ca} \times \vec{\mathbf{V}}\_{cp} \\ 
\vec{\mathbf{n}}\_3&=\vec{\mathbf{V}}_{ab} \times \vec{\mathbf{V}}\_{ap} 
\end{aligned}
$$


The area of a triangle can be calculated as half of the cross product of the vectors forming two sides of the triangle. We can thus write the areas as follows:

$$
\begin{aligned}
A&=\frac{1}{2}|\vec{\mathbf{n}}| \\
A_1&=\frac{1}{2}|\vec{\mathbf{n}}_1| \\
A_2&=\frac{1}{2}|\vec{\mathbf{n}}_2| \\
A_3&=\frac{1}{2}|\vec{\mathbf{n}}_3|
\end{aligned} 
$$

These always give positive values for area, but we can check whether the area is positive or negative by comparing the normal vector of the subtriangle with the overall triangle using a dot product. 
The result will be positive if the normal vectors are in the same direction or negative if they are in opposite directions.
The way I have defined each of the normal vectors above ensures the correct orientation.
For example,

$$
\frac{\vec{\mathbf{n}}\cdot\vec{\mathbf{n}}_1}{|\vec{\mathbf{n}}||\vec{\mathbf{n}}_1|}=% 
                    \begin{cases} 
                    1 & \text{if P is inside the triangle} \\ 
                    -1 & \text{if P is outside the triangle, across edge b-c} \end{cases} 
$$

Multiplying the area ratio by the corresponding dot product term gives the barycentric coordinates.

$$
\begin{aligned} 
    \xi_1&=\frac{\vec{\mathbf{n}}\cdot\vec{\mathbf{n}}_1}{|\vec{\mathbf{n}}|^2} \\ 
    \xi_2&=\frac{\vec{\mathbf{n}}\cdot\vec{\mathbf{n}}_2}{|\vec{\mathbf{n}}|^2} \\ 
    \xi_3&=\frac{\vec{\mathbf{n}}\cdot\vec{\mathbf{n}}_3}{|\vec{\mathbf{n}}|^2}  
\end{aligned}
$$

### Extending to Three Dimensions

This same principle applies to a tetrahedron in three dimensions. The barycentric coordinates of a tetrahedron are based on the volume of subtetrahedra, and the coordinates will have four components. The tetrahedron below, formed by vertices $a$, $b$, $c$, and $d$ can be subdivided by inserting a point $p$.

![Subdivided tetrahedron](/assets/images/barycentric/barycentric_tetrahedron.png){: .align-center}

The volume of a tetrahedron can be calculated with a scalar triple product.
Let $V$ is the total volume of the tetrahedron, $V_a$ be the total volume of the subtetrahedron opposite vertex $a$, and similarly for the others.
Then the volumes of each subtetrahedron and the total volume are given by the following expressions.

$$
\begin{aligned} 
    V_a&=\frac{1}{6}(\vec{\mathbf{V}}_{bp}\cdot(\vec{\mathbf{V}}_{bd}\times \vec{\mathbf{V}}_{bc})) \\ 
    V_b&=\frac{1}{6}(\vec{\mathbf{V}}_{ap}\cdot(\vec{\mathbf{V}}_{ac}\times \vec{\mathbf{V}}_{ad})) \\ 
    V_c&=\frac{1}{6}(\vec{\mathbf{V}}_{ap}\cdot(\vec{\mathbf{V}}_{ad}\times \vec{\mathbf{V}}_{ab})) \\  
    V_d&=\frac{1}{6}(\vec{\mathbf{V}}_{ap}\cdot(\vec{\mathbf{V}}_{ab}\times \vec{\mathbf{V}}_{ac})) \\  
    V&=\frac{1}{6}(\vec{\mathbf{V}}_{ab}\cdot(\vec{\mathbf{V}}_{ac}\times \vec{\mathbf{V}}_{ad}))  
\end{aligned}
$$

The barycentric coordinates are then given by the ratio of each subtetrahedron to the total volume.

$$
\begin{aligned} 
    \xi_1&=\frac{V_a}{|V|} \\ 
    \xi_2&=\frac{V_b}{|V|} \\ 
    \xi_3&=\frac{V_c}{|V|} \\ 
    \xi_4&=\frac{V_d}{|V|}  
\end{aligned}
$$

## Use Cases
### Barycentric Interpolation
Triangular and tetrahedron-based meshes are prevalent in CFD and finite element codes.
Often, we want to know the value of some property between nodes on the mesh.
This is simple with barycentric interpolation.
If the value at node $i$ on your element is $Q_i$, then the value at a point inside that element is given by the sum of that property at each node, weighted by the corresponding barycentric coordinate.

$$
Q = \sum_{i=1}^{n}Q_i \xi_i
$$

Note that this works for both 2D and 3D meshes.

### Locating the Element that Contains a Point
As I mentioned earlier, if any of the barycentric coordinate components are negative, then the point lies outside of your element. 
Even cooler is the fact that the point will lie on the side of the element with the negative component. 
This enables us to create a semi-intelligent search algorithm. 
We can take an initial guess at which element the point is in. 
If it's not the right element, we simply check the element across the side with the negative barycentric component. 
If we continue this, we will eventually find an element that gives us all positive barycentric components. 
That element contains the point.

There are many other uses for barycentric coordinates. If you have used barycentric coordinates for something interesting, let me know and I will add it to this list. 

## Source Code

On my [GitHub](https://github.com/csimpson88), I have shared a simple [barycentric library](https://github.com/csimpson88/barycentric) that I wrote in pure C. Feel free to check it out and use it in your own projects.