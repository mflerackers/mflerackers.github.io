---
title: Polygons
math: true
excerpt: "What is the winding order of a polygon. How to determine the area. How to know whether a points lies in a polygon. How to determine whether a polygon is convex or concave. How to perform collision detection and response between polygons."
---

| : ---- : |
| [Previous - Triangles](6-triangles.html) | Next - Boxes |

## Polygon definition

A polygon is a closed shape defined by three or more points. Polygons, like triangles, have a winding order defined by the order of the points. Unlike triangles however, polygons come in two additional flavors, convex or concave. Convex polygons, to which triangles and rectangles belong, have the property that if you take any two points belonging to the polygon, that all points on the line between those points also belong to the polygon. For a star polygon, which is convex, this is not the case for example.

### Polygon area and winding order

The winding order of a polygon can be obtained by calculating a sum of cross products, which give us double the signed area of the polygon. The formula you find under the name shoelace formula or Gauss's area formula usually comes under the following form

$$
\frac{1}{2}\left\lvert\sum\limits_{i=1}^{n-1}x_iy_{i+1} + x_ny_1 - \sum\limits_{i=1}^{n-1}x_{i+1}y_i - x_1y_n\right\lvert
$$

and gives us the area of the polygon. If we want the winding order, we don't need the absolute value or division, thus we can look at

$$
\sum\limits_{i=1}^{n-1}x_iy_{i+1} + x_ny_1 - \sum\limits_{i=1}^{n-1}x_{i+1}y_i - x_1y_n
$$

Which, if we switch some terms becomes

$$
\sum\limits_{i=1}^{n-1}x_iy_{i+1} - \sum\limits_{i=1}^{n-1}x_{i+1}y_i + x_ny_1 - x_1y_n
$$

Which, after grouping some terms becomes

$$
\sum\limits_{i=1}^{n-1}{(x_iy_{i+1} - x_{i+1}y_i)} + x_ny_1 - x_1y_n
$$

Which is is clearly several cross products

$$
\sum\limits_{i=1}^{n-1}cross(P_i, P_{i+1}) + cross(P_n, P_1)
$$

First of all, why the separate cross product? Well, we take two points in sequence every time, and at the end we need to wrap around, as the last point uses the first.

Next, how does this formula differ with the one we got for triangles? At first glance, it looks familiar but different. Both use cross products. Where the triangle formula used just one cross product, this one will use three for a triangle. And where the triangle formula used vectors originating from one of the points towards the remaining points, this formula uses vectors originating from the origin towards the points.

But do we get the same result for a triangle? Well let's calculate.

Our triangle formula used to get the winding order was $$\vec{AB}\times\vec{AC}$$. So if A, B and C are written as $$(x_1, y_1)$$, $$(x_2, y_2)$$ and $$(x_3, y_3)$$ we get that

$$
\vec{AB} = (x_2-x_1, y_2-y_1) \\
\vec{AC} = (x_3-x_1, y_3-y_1)
$$

Calculating the cross product gives us

$$
\vec{AB}\times\vec{AC}=(x_2-x_1)(y_3-y_1) - (y_2-y_1)(x_3-x_1)
$$

Writing out the products gives us

$$
x_2y_3-x_2y_1-x_1y_3+x_1y_1 - y_2x_3+y_2x_1+y_1x_3-y_1x_1
$$

The $$+x_1y_1$$ and $$-y_1x_1$$ cancel each other out so we get

$$
x_2y_3-x_2y_1-x_1y_3 - y_2x_3+y_2x_1+y_1x_3
$$

Now let's look at the new formula. For a 3 point polygon like our triangle we get

$$
x_1y_2+x_2y_3+x_3y_1-x_2y_1-x_3y_2-x_1y_3
$$

Which is exactly the same as our previous formula.

Now, the formula we are going to use in our code has yet another form

$$
\sum\limits_{i=1}^{n}(x_{i+1}+x_i)(y_{i+1}-y_i)
$$

Where n+1 wraps back to 1. When we write out this formula for our triangle we get

$$
(x_2+x_1)(y_2-y_1) + (x_3+x_2)(y_3-y_2) + (x_1+x_3)(y_1-y_3)
$$

Writing this out gives

$$
x_2y_2 - x_2y_1 + x_1y_2 - x_1y_1 + x_3y_3 - x_3y_2 + x_2y_3 - x_2y_2 + x_1y_1 - x_1y_3 + x_3y_1 - x_3y_3
$$

Eliminating opposing pairs gives us

$$
-x_2y_1 + x_1y_2 - x_3y_2 + x_2y_3 - x_1y_3 + x_3y_1
$$

Which gives our original formula again. This form is used most of the time as it takes an addition, subtraction and multiplication rather than two multiplications and a subtraction, which is generally cheaper to do. Though if you use SIMD instructions, cross products is most likely the better choice.

```lua
function polygonWindingOrder(points)
    if #points < 6 then return nil end

    local sum = (points[1] + points[#points-1]) * (points[2] - points[#points])
    for i=1, #points-3, 2 do
        sum = sum + (points[i+2] + points[i]) * (points[i+3] - points[i+1])
    end

    if sum < 0 then
        return "ccw"
    else
        return "cw"
    end
end

function polygonArea(points)
    if #points < 6 then return nil end

    local sum = (points[1] + points[#points-1]) * (points[2] - points[#points])
    for i=1, #points-3, 2 do
        sum = sum + (points[i+2] + points[i]) * (points[i+3] - points[i+1])
    end

    return math.abs(sum) * 0.5;
end
```

### Convex or concave

Another important property of polygons is whether they are convex or concave. A convex polygon has the property that if you take any two points, the line between those points falls entirely within the polygon. Thus a rectangle or hexagon is convex, while a star polygon is not.

Whether a polygon is convex or concave has implications in use. Any convex polygon can for example be much easier split into triangles, while concave polygons which are not stars need more complex steps. The same goes for collision tests.

So how can we know whether a given polygon is convex or concave? Two adjacent sides of a convex polygon will always turn towards the polygon, they go around the polygon, not alternating towards and away like a star for example. Thus the angle between each two sides can tell us whether the polygon is convex or not. Instead of the angle we only need to know the sign of the sine however, as we only need to know whether all angles are greater or smaller than 0, which maps to a positive or negative sine. Since we are only interested in the sign and can disregard the magnitude, we can use the cross product.

So our final algorithm calculates the cross product between every two adjacent sides. We can exit early if the sign switches as we know the polygon will be concave in that case. If we pass the whole polygon with the same sign, we have a convex polygon.

```lua
function polygonType(points)
    if #points < 6 then return nil end

    local v1x, v1y = points[#points-1] - points[1], points[#points] - points[2]
    local v2x, v2y = points[3] - points[1], points[4] - points[2]
    local sign = v1x * v2y - v1y * v2x
    for i=1, #points-5, 2 do
        v1x, v1y = -v2x, -v2y
        v2x, v2y = points[i+4] - points[i+2], points[i+5] - points[i+3]
        if (sign * (v1x * v2y - v1y * v2x)) < 0 then
            return "concave"
        end
    end

    return "convex"
end
```

Drag the vertices below to see how the cross product of adjacent edges changes depending on whether the vertex makes the polygon convex or concave. Green is used for a negative cross product, while red is used for a positive cross product.

<p data-height="400" data-theme-id="0" data-slug-hash="ajPWYK" data-default-tab="result" data-user="mflerackers" data-pen-title="Convex or concave" class="codepen">See the Pen <a href="https://codepen.io/mflerackers/pen/ajPWYK/">Convex or concave</a> by Marc (<a href="https://codepen.io/mflerackers">@mflerackers</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## Point in polygon

One of the ways to check whether a point lies within a polygon is using the same method as we used with triangles, we look at the sign of cross products. However that are a lot of cross products when our polygon contains more than a few points. Of course, if we know the winding order and that the polygon is convex, we can reduce the amount of cross products. But since we want a general solution, we're going to look at a faster way of doing it for all polygons.

We still do a check for every edge, but we only look for edges which cross our y coordinate. When they do, we find the x coordinate on the edge and check whether it is larger than our x. The idea is that we count how many crossings of edges lie on the right side. If it is odd, we are inside the polygon, if it is even, we are outside the polygon.

The test to look whether y lies between y1 and y2, is written as (y1 >= y) != (y2 >= y). This test only passes if one of them is true, not both. So if (y1 >= y and y2 < y) or (y1 < y and y2 >= y). Moreover, note that it doesn't pass when y1 == y2, which is important for the next test in order to avoid a division by zero.

To obtain x' of the intersection at y'=y, to test whether x is smaller than x', we can use the vector equation of the line.

$$
(x', y') = (x_1, y_1) + t * (x_2 - x_1, y_2 - y_1)
$$

or

$$
x' = x_1 + t * (x_2 - x_1) \\
y' = y_1 + t * (y_2 - y_1)
$$

Since we know y' is equal to y, we can get t

$$
t = \frac{(y-y_1)}{(y_2-y_1)}
$$

Filling this t into the equation for x' we get

$$
x' = x_1 + \frac{(y-y_1)}{(y_2-y_1)} * (x_2 - x_1)
$$

```lua
function Polygon:contains(x, y)
    if #self.points < 6 then return false end
    local x1, y1, x2, y2 = self.points[#self.points-1], self.points[#self.points], self.points[1], self.points[2]
    local c = false
    for i = 1, #self.points-1, 2 do
        if ((y1 >= y) ~= (y2 >= y)) and (x <= (x2 - x1) * (y - y1) / (y2 - y1) + x1) then
            c = not c
        end
        x1, y1 = x2, y2
        x2, y2 = self.points[i+2], self.points[i+3]
    end
    return c
end
```

You can see the result below. Blue is the polygon, red is the point that is being tested. Purple are points for which x is greater than the x of the red point. Green are points for which x is smaller than the x of the red point. When the amount of purple points is odd, the red point lies in the blue polygon. You can move both the red point and the polygon's vertices to see what the result is for different setups.

<p data-height="402" data-theme-id="0" data-slug-hash="KBBRrX" data-default-tab="result" data-user="mflerackers" data-pen-title="Point in polygon" class="codepen">See the Pen <a href="https://codepen.io/mflerackers/pen/KBBRrX/">Point in polygon</a> by Marc (<a href="https://codepen.io/mflerackers">@mflerackers</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## Polygon collision detection

The following collision detection method only works with convex polygons. So if you have concave polygons, you either need to triangulate them or split them into convex polygons.

The collision detection algorithm we use, called SAT or separating axis theorem, is quite simple. Remember that we said that we could use vector projection to do collision detection? What we do is project both polygons onto a line. If the resulting line segments don't overlap, the polygons don't collide.

Of course if they do overlap, it doesn't mean that they collide, there might be another line on which the projected polygons don't overlap.

Luckily there's a limit to the amount of projections we have to do. We only need to project onto the distinct normals of the polygons.

So the algorithm is basically

```lua
function projectPolygon(points, x, y, nx, ny)
  local min = (points[1]-x)*nx+(points[2]-y)*ny
  local max = min
  local s
  for i = 3, #points, 2 do
    s = (points[i]-x)*nx+(points[i+1]-y)*ny
    if s < min then min = s end
    if s > max then max = s end
  end
  return min, max
end

function polygonCollidesWith(points1, points2)
  -- We project both polygons on all normals of polygon 1
  local x1, y1 = points1[#points1-1], points1[#points1]
  local x2, y2 = points1[1], points1[2]
  local nx, ny, min1, max1, min2, max2
  -- For every edge of points1
  for i = 1, #points1, 2 do
    -- Get edge normal by rotating the vector [x2-x1, y2-y1] by 90 degrees
    nx, ny = y2-y1, x1-x2;
    -- Project both polygons onto the normal
    min1, max1 = projectPolygon(points1, x2, y1, nx, ny)
    min2, max2 = projectPolygon(points2, x2, y1, nx, ny)
    -- If there is no overlap between the ranges, there is no collision
    if max1 <= min2 or min1 >= max2 then
      return false
    end
    x1, y1 = x2, y2
    x2, y2 = points1[i+2], points1[i+3]
  end
  return true;
end

function polygonsCollide(points1, points2)
  return polygonCollidesWith(points1, points2) and polygonCollidesWith(points2, points1)
end
```

Note that we don't need a vector projection, a scalar projection is sufficient. The scalar projection of a point onto a vector gives us the position of the projected point as a one dimensional coordinate on the line defined by the vector. Note that we don't even need to divide by the dot product of the vector we project onto as a scale will not change whether the ranges overlap or not.

So we take the unscaled scalar projection of each point in the polygon, and we only keep the minimum and maximum coordinate of these projections. This gives us our polygon flattened into a one dimensional range, or a line segment if we would project it back to 2D.

We do this for both polygons and check whether the ranges overlap, if they don't, the polygons don't collide. If they do, we check the next normal.

If we can't find a normal for which the projections overlap, the polygons don't collide.

You can test the algorithm below, by dragging vertices, edges or polygons. Note though that both your polygons should be convex.

<p data-height="383" data-theme-id="0" data-slug-hash="ajXwOa" data-default-tab="result" data-user="mflerackers" data-pen-title="Polygon collision" class="codepen">See the Pen <a href="https://codepen.io/mflerackers/pen/ajXwOa/">Polygon collision</a> by Marc (<a href="https://codepen.io/mflerackers">@mflerackers</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## Collision resolution

While there is use for checking collisions without resolving the collision, think for example dragging and dropping a shape, like a puzzle piece, onto a hole, in many cases we do need to restore the polygons to a state where they don't collide.

The idea is to use the overlap we found previously to displace one of the polygons. This means we need to keep the smallest overlap (since we want the smallest correction possible) as well as the direction vector in order to do the displacement.

Finally, we need to check whether our displacement vector points in the correct direction. Given that we might have two normals giving the same orientation but the opposite sign, a square for example, we need to check for this. We can use the dot product here. If the dot product of the displacement vector and the vector indicating the direction in which the displaced polygon lies with respect to the static polygon is smaller than 0, the vectors point in the opposite direction and we need to flip the displacement vector.

How do we determine the vector pointing from the static polygon to the displaced polygon? We take the centers of both polygons, and we subtract the center of the static polygon from the center of the displaced polygon.

```lua
function polygonCollidesWithV(points1, points2)
  -- We project both polygons on all normals of polygon 1
  local x1, y1 = points1[#points1-1], points1[#points1];
  local x2, y2 = points1[1], points1[2];
  local nx, ny, min1, max1, min2, max2;
  local overlap, smallestOverlap, length, overlapX, overlapY;
  -- For every edge of points1
  for i = 1, #points1, 2 do
    -- Get edge normal by rotating the vector [x2-x1, y2-y1] by 90 degrees
    nx, ny = y2-y1, x1-x2
    -- Project both polygons onto the normal
    min1, max1 = projectPolygon(points1, x2, y1, nx, ny);
    min2, max2 = projectPolygon(points2, x2, y1, nx, ny);
    -- If there is no overlap between the ranges, there is no collision
    if max1 <= min2 or min1 >= max2 then
      return false
    end
    overlap = math.min(max1, max2) - math.max(min1, min2);
    -- Our scalar projection wasn't scaled yet
    length = nx*nx+ny*ny;
    overlap = overlap/ length;
    -- And we need to take the length of the vector into account for the overlap length
    length = math.sqrt(length);
    -- Record smallest overlap
    if not smallestOverlap or overlap * length < smallestOverlap then
      smallestOverlap = overlap * length;
      overlapX = nx*overlap;
      overlapY = ny*overlap;
    end
    x1, y1 = x2, y2;
    x2, y2 = points1[i+2], points1[i+3];
  end
  return smallestOverlap, overlapX, overlapY;
end

function polygonsCollideV(points1, points2)
  local overlap1, x1, y1 = polygonCollidesWithV(points1, points2)
  local overlap2, x2, y2 = polygonCollidesWithV(points2, points1)
  if overlap1 and overlap2 then
    if overlap1 < overlap2 then
      return overlap1, x1, y1
    else
      return overlap2, x2, y2
    end
  else
    return false
  end
end
```

Note that unlike before, we normalize the vectors, this is because otherwise we need to scale the radius of the circle into normal space, to be able to compare it with the other projections in normal space. To avoid this, we use a normalized vector, which doesn't cost us more, as the alternative is to multiply the radius with the squared length of the vector and dividing it by the length of the vector.

You can see how it works below. The green line is the vector from the center of the static polygon towards the displaced polygon. The red line is the vector which will move the displaced polygon away from the static polygon.

<p data-height="420" data-theme-id="0" data-slug-hash="VBgOeQ" data-default-tab="result" data-user="mflerackers" data-pen-title="Polygon collision resolution" class="codepen">See the Pen <a href="https://codepen.io/mflerackers/pen/VBgOeQ/">Polygon collision resolution</a> by Marc (<a href="https://codepen.io/mflerackers">@mflerackers</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

What if both polygons are moving? Then the displacement will be distributed to both polygons according to their mass. This will be explained once we get to physics.

## Circles

While being able to detect collisions between convex polygons is nice, we often meet circles in games. Wheels, rolling stones or characters, they are all very round. Now, while a circle is most of the time drawn as a polygon approximating the actual circle, we don't really need to use such an expensive polygon to test with. We can use approximately the same algorithm as before.

Projecting the circle is very simple, we only project the center point, and subtract and add the radius to get the minimum and maximum.

But what normal should we use? There are an infinite number of normals for a circle, and still far too many in an approximation.

We are in luck again, as the only axis we need to test for is the one defined by the center of the circle and the closest point in the polygon.

You can test the result below by dragging the circle towards the polygon. The additional circle axis comes into play when the circle is close to one of the vertices. Not that if the circle is too deep into the polygon, it needs several steps to get pushed out of it. This is because the maximum overlap, and thus displacement, will be the diameter of the circle.

<p data-height="384" data-theme-id="0" data-slug-hash="rrRLNG" data-default-tab="result" data-user="mflerackers" data-pen-title="Polygon circle collision" class="codepen">See the Pen <a href="https://codepen.io/mflerackers/pen/rrRLNG/">Polygon circle collision</a> by Marc (<a href="https://codepen.io/mflerackers">@mflerackers</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>