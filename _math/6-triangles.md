---
title: Triangles
math: true
---

| : ---- : |
| [Previous - Lines](5-lines.html) | Next - Polygons |

## Triangles

A triangle is a polygon with 3 points. The order of the points indicates the winding order, clockwise or counter clockwise.

### Baricentric coordinates

The baricentric coordinates of a point in a triangle is a triplet where each dimension indicates the relative distance to one of the points. If this sounds too complex, let us look at some examples.

The baricentric coordinates of A are $$(1,0,0)$$ as A is closest to A and furthest from the other points.

For a point in the middle of the line AB the baricentric coordinates would be $$(0.5,0.5,0)$$ as the point is equally far from A as from B, and as far as is posible from C while staying inside the triangle.

The center will have as barycentric coordinates $$(\frac{1}{3},\frac{1}{3},\frac{1}{3})$$.

To reconstruct the point, we just multiply each point of the triangle with its corresponding barycentric coordinate, and add the resulting terms.

$$
P=aA+bB+cC
$$

### Triangle center

Given the three points of the triangle A, B, C, we can calculate the center of the triangle as $$\frac{A+B+C}{3}$$.

### Point in triangle

#### Using barycentric coordinates

If we know that the barycentric coordinates are all within 0 and 1, and their sum is not greater than 1, we can say that the point is within the triangle.

```lua
function pointInTriangle2(px,py,ax,ay,bx,by,cx,cy)

    -- Check ABxAP relative to ABxAC
    local abx, aby = vector(ax, ay, bx, by)
    local acx, acy = vector(ax, ay, cx, cy)
    local apx, apy = vector(ax, ay, px, py)
    local denom = cross(abx, aby, acx, acy)

    if denom < 0.0001 and denom > -0.0001 then return false end

    local numer = cross(abx, aby, apx, apy)
    local c = numer / denom

    if c < 0 or c > 1 then return false end

    -- Check APxAC relative to ABxAC
    numer = cross(apx, apy, acx, acy)
    local b = numer / denom

    if b < 0 or b > 1 then return false end

    return c + b <= 1
end
```

#### Using edge tests

Another way to know whether a point p is inside a triangle is to check on which side the point lies for each edge. If the point lies on the side of the triangle for each side, it lies within the triangle.

How do we test this? Fairly simple. We take the vector AB and the vector AC and calculate the cross product. Then we do the same with AB and AP. Remember that for the cross product, the order in which we take vectors will decide the sign of the result. If we take AB x AC, we know which sign gives a point on the correct side of AB. If AB x AP matches that sign, P lies on the same side of the triangle. If we repeat this for the other two edges, we can determine whether P lies inside the triangle.

```lua
local function pointInTriangle(px,py,ax,ay,bx,by,cx,cy)
    
    -- Check P relative to AB
    local abx, aby = vector(ax, ay, bx, by)
    local acx, acy = vector(ax, ay, cx, cy)
    local apx, apy = vector(ax, ay, px, py)
    local tside = cross(abx, aby, acx, acy)
    local pside = cross(abx, aby, apx, apy)
    if (tside >=0 and pside < 0) or (tside < 0 and pside >= 0) then
      return false
    end

    -- Check P relative to AC
    pside = cross(apx, apy, acx, acy)
    if (tside >=0 and pside < 0) or (tside < 0 and pside >= 0) then
      return false
    end

    -- Check P relative to CB
    local cax, cay = -acx, -acy
    local cbx, cby = vector(cx, cy, bx, by)
    local cpx, cpy = vector(cx, cy, px, py)
    tside = cross(cbx, cby, cax, cay)
    pside = cross(cbx, cby, cpx, cpy)
    if (tside >=0 and pside < 0) or (tside < 0 and pside >= 0) then 
      return false
    end

    return true
end
```

## Triangle equations

### Right-angled triangle

A common, and for some dreaded, theme concerning triangles is the properties of a right-angled triangle. Given an angle and a side, calculate the remaining sides for example. However, we don't need to learn or remember anything we haven't covered yet.

Let's start by laying the triangle in our favorite configuration. First we rotate the triangle towards the x axis.

![rotate_triangle](/assets/rotate_triangle.png)

Then we scale it so that the length of c is 1.

![rotate_scale_triangle](/assets/rotate_scale_triangle.png)

We can see that if the length of c was 1, then a would be equal to the cosine, while b would be equal to the sine. Since c in most situations is not 1, we need to scale the cosine and sine by the length of c, thus

$$
a=c*cos(\alpha)\\
b=c*sin(\alpha)
$$

That's all there is to it. Don't have $$\alpha$$ but $$\beta$$? Just turn the triangle, or use $$\alpha=90-\beta$$, since the sum of the angles is 180 degrees and one of them is 90. Don't have c? Just divide both sides of the equation by cosine and sine respectively.

$$
c=\frac{a}{cos(\alpha)} \\
c=\frac{b}{sin(\alpha)}
$$

Also since we know that $$cos(\alpha)^2+sin(\alpha)^2 =1$$, since the dot product of the vector with itself gives us its squared length, which is equal to 1, we can write

$$
c^2*cos(\alpha)^2+c^2*sin(\alpha)^2 =c^2
$$

by multiplying each side with $$c^2$$. If we regroup the terms we get 

$$
(c*cos(\alpha))^2+(c*sin(\alpha))^2 =c^2
$$

Which gives us

$$
a^2+b^2 =c^2
$$

Unlike learning rules, this is something you can reconstruct any time you need it.

### General triangle

