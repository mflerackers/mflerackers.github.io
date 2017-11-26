---
layout: default
title: Lines
math: true
---

[Previous - Matrices](4-matrices.html)

# [Math for game development](../) : Lines

## Lines

A line can be defined using any two points lying on the line. Let's call the points on our line $$a$$ and $$b$$. If we draw a line through these two points, we end up with our original line.

Now let's build a vector from these points going from $$a$$ to $$b$$ called $$\vec{ab}$$.

$$
\vec{ab}=b-a
$$

Our line can now be defined using the point $$a$$ and the vector $$\vec{ab}$$.

$$
p=a+s*\vec{ab}
$$

For every scalar s the point p will be a point on our line.

This is the vector equation of a line. However most people might be more acquainted with the parametric equation of a line

$$
y=ax+b
$$

Where $$b$$ is the offset from the x-origin and $$a$$ is the slope (amount of $$y$$ units for one $$x$$ unit). Both equations describe the same function and can be derived from each other. We can write the parametric equation as two functions of $$t$$ as follows

$$
x=t
$$

$$
y=at+b
$$

The function of x can be written as a line equation as well by using an offset of 0 and a slope of one.

$$
x=1t+0
$$

$$
y=at+b
$$

If we write this in vector form we get

$$
(x,y)=(1,a)t+(0,b)
$$

Which is our vector equation of a line, where the point of the line is chosen where the line crosses the x axis at 0 and the direction vector is divided by its x coordinate to make the x coordinate 1.

### Line intersection

Given two lines through a, b and c, d respectively.

If these lines intersect, then there is a point p for which there is both a scalar t and a scalar s for which the following equations are equal to p.

$$
p=a+s*\vec{ab}
$$

$$
p=c+t*\vec{cd}
$$

Since p is the same in this case we can write

$$
a+s*\vec{ab}=c+t*\vec{cd}
$$

If we subtract a from both sides we get

$$
a-a+s*\vec{ab}=c-a+t*\vec{cd}
$$

The point $$a$$ subtracted from $$$c$ gives us the vector $$\vec{ac}$$

$$
s*\vec{ab}=\vec{ac}+t*\vec{cd}
$$

Now we are going to use the cross product with $$\vec{cd}$$ to get rid of the term $$t*\vec{cd}$$, as $$\vec{cd}\times\vec{cd}$$ is 0.

$$
s*\vec{ab}\times\vec{cd}=\vec{ac}\times\vec{cd}+t*\vec{cd}\times\vec{cd}
$$

Erasing the 0 term gives us

$$
s*\vec{ab}\times\vec{cd}=\vec{ac}\times\vec{cd}
$$

This means s is

$$
\frac{\vec{ac}\times\vec{cd}}{\vec{ab}\times\vec{cd}}=s
$$

Before we go on, let's note that $$\vec{ab}\times\vec{cd}$$ can be 0, and we shouldn't divide by 0. This cross product is zero when our lines are parallel and thus have either no intersection, or are collinear and are thus intersecting everywhere. Remember, the cross product gives us the sine of the angle (multiplied with a factor if the vectors are not normalized). The sine is 0 in the cases of 0 and 180 degrees. If there's an angle of 0 or 180 degrees between the lines, they are definitely parallel, or collinear.

Now that we know s, we can calculate the point p from the formula $$p=a+s*\vec{ab}$$. This gives us the intersection point. Note that if the lines are collinear, we still return nil, as we can't tell what point to return when everything collides.

~~~ Lua
function intersectLines(ax, ay, bx, by, cx, cy, dx, dy)
    local abx, aby = bx - ax, by - ay           -- ab
    local cdx, cdy = dx - cx, dy - cy           -- cd
    local s = cross(abx, aby, cdx, cdy)         -- ab x cd
    if s < 0.00001 and s > -0.00001 then
        -- Collinear if ac x cd == 0, parallel otherwise
        return nil
    end
    s = cross(cx - ax, cy - ay, cdx, cdy) / s   -- ac x cd / ab x cd
    return ax + s * abx, ay + s * aby           -- a + s * ab
end
~~~

### Point on line

Let's say we have the line through points a and b, and we need to know whether c is on the same line. Remember how we tested for parallel lines using the cross product? That is exactly what we need now. If we build a vector from a to c called ac, and cross it with the vector ab, the result is 0 if c lies on ab.

~~~ Lua
function pointOnLine(ax, ay, bx, by, cx, cy)
    local abx, aby = bx - ax, by - ay -- ab
    local acx, acy = cx - ax, cy - ay -- ac
    local t = cross(abx, aby, acx, acy) -- ab x ac
    return t < 0.00001 and t > -0.00001
end
~~~
## Line segments

Sometimes our lines don't go to infinity in both directions, but have given boundaries. Line segments are not much different in usage, except for some extra testing to make sure we are actually still on the line segment. Given a segment between points $$a$$ and $$b$$, we saw we could define the line as

$$
p=a+s*\vec{ab}
$$

We can see that if s is 0, p is equal to a, while if s is 1, p is equal to b. Any value of s not between 0 and 1 gives a point outside of the line segment.

### Line segment intersection

#### Line segment and line

Given what we know from line intersection, intersecting a line or line segment with another line is not so hard. We only need to be sure that the intersection point is actually within the segment(s). This is simple if we have s for the point on the segment. If s is outside the interval [0,1], the point is outside the range of the segment.

~~~ Lua
function intersectLineSegmentLine(ax, ay, bx, by, cx, cy, dx, dy)
    local abx, aby = bx - ax, by - ay           -- ab
    local cdx, cdy = dx - cx, dy - cy           -- cd
    local s = cross(abx, aby, cdx, cdy)         -- ab x cd
    if s < 0.00001 and s > -0.00001 then
        -- Collinear if ac x cd == 0, parallel otherwise
        return nil
    end
    s = cross(cx - ax, cy - ay, cdx, cdy) / s   -- ac x cd / ab x cd
    if s < 0 or s > 1 then
        return nil
    end
    return ax + s * abx, ay + s * aby           -- a + s * ab
end
~~~

#### Line segment and line segment

To know whether our intersection point lies on the other segment, we need to calculate $$s$$ like we calculated $$t$$ before.
$$
a+s*\vec{ab}=c+t*\vec{cd}
$$
If we subtract c from both sides we get
$$
a-c+s*\vec{ab}=c-c+t*\vec{cd}
$$
The point $$c$$ subtracted from $$a$$ gives us the vector $$\vec{ca}$$
$$
\vec{ca}+s*\vec{ab}=t*\vec{cd}
$$
Now we use the cross product with $$\vec{ab}$$ to get rid of the term $$s*\vec{ab}$$, as $$\vec{ab}\times\vec{ab}$$ is 0.
$$
\vec{ca}\times\vec{ab}+s*\vec{ab}\times\vec{ab}=t*\vec{cd}\times\vec{ab}
$$
Erasing the 0 term gives us
$$
\vec{ca}\times\vec{ab}=t*\vec{cd}\times\vec{ab}
$$
This means that t is
$$
s=\frac{\vec{ca}\times\vec{ab}}{\vec{cd}\times\vec{ab}}
$$
Now we already have $$\vec{ab}\times\vec{cd}$$, and we know that $$\vec{cd}\times\vec{ab}=-\vec{ab}\times\vec{cd}$$ because switching vectors in a cross product gives use the cross product of the other angle, as $$sin(\alpha)=-sin(-\alpha)$$. We also have $$\vec{ac}$$, while here we require $$\vec{ca}$$. If we invert the vector $$\vec{ca}$$ to the vector $$\vec{ac}$$ in $$\vec{ca}\times\vec{ab}$$ however, we invert the sign once more, as $$\vec{ca}\times\vec{ab}=-\vec{ac}\times\vec{ab}$$.  So we get

$$
s=\frac{\vec{ca}\times\vec{ab}}{\vec{cd}\times\vec{ab}}=\frac{-\vec{ac}\times\vec{ab}}{-\vec{ab}\times\vec{cd}}=\frac{\vec{ac}\times\vec{ab}}{\vec{ab}\times\vec{cd}}
$$

Using this we can efficiently write the checks to see if our point lies within the other segment as well.

```Lua
function intersectLineSegments(ax, ay, bx, by, cx, cy, dx, dy)
    local abx, aby = bx - ax, by - ay           -- ab
    local cdx, cdy = dx - cx, dy - cy           -- cd
    local abxcd = cross(abx, aby, cdx, cdy)     -- ab x cd
    if abxcd < 0.00001 and abxcd > -0.00001 then
        -- Collinear if ac x cd == 0, parallel otherwise
        return nil
    end
    local acx, acy = cx - ax, cy - ay
    s = cross(acx, acy, cdx, cdy) / abxcd       -- ac x cd / ab x cd
    if s < 0 or s > 1 then
        return nil
    end
    local t = cross(acx, acy, abx, aby) / abxcd -- ac x ab / ab x cd
    if t < 0 or t > 1 then
        return nil
    end
    return ax + s * abx, ay + s * aby           -- a + s * ab
end
```

### Point on line segment

We first need to test whether the point lies on the line using the cross product as we did before for infinite lines.

If the point is on the line, we need to test whether it lies between the segment's boundaries. We can do this second step partly by calculating the dot product, like when we did vector projection. If the dot product is smaller than 0, the point lies before the start of the segment, as the angle is greater than 90 degrees in that case.

To be sure that the point doesn't lie past the end of the segment, we can look whether the dot product of $$\vec{ac}$$ with itself is smaller than the dot product of $$\vec{ab}$$ with itself, as the dot product gives us the length squared.

~~~ Lua
function pointOnLineSegment(ax, ay, bx, by, cx, cy)
    local abx, aby = bx - ax, by - ay        -- ab
    local acx, acy = cx - ax, cy - ay        -- ac
    local t = cross(abx, aby, acx, acy)      -- ab x ac
    if t > 0.00001 or t < -0.00001 then
        return false
    end
    t = dot(abx, aby, acx, acy)              -- ab . ac
    if t < 0 then
        return false
    end
    t = dot(acx, acy, acx, acy)            -- ac . ac
    return t <= dot(abx, aby, abx, aby)    -- ab . ab
end
~~~
## Rays

A ray is a segment which is bounded at one side, the origin, while it stretches to infinity one the other side. Rays have various usages. As rays of light to determine highlights for example. The dot product of the ray with a surface normal gives a simple but fast diffuse shading. Or to determine where shadows fall by determining walls whose normals point towards the light and projecting those walls further along the ray. Sometimes a line segment is used as a ray, like when determining visibility from the player's position when there is a "fog of war", as the ray needs a limited range.

### Ray intersection

Intersecting a line with a ray is similar to a segment except that we only need to test for t < 0, as the ray is only bounded at the origin.

~~~ Lua
function intersectRayLine(ax, ay, bx, by, cx, cy, dx, dy)
    local abx, aby = bx - ax, by - ay           -- ab
    local cdx, cdy = dx - cx, dy - cy           -- cd
    local s = cross(abx, aby, cdx, cdy)         -- ab x cd
    if s < 0.00001 and s > -0.00001 then
        -- Collinear if ac x cd == 0, parallel otherwise
        return nil
    end
    s = cross(cx - ax, cy - ay, cdx, cdy) / s   -- ac x cd / ab x cd
    if s < 0 then
        return nil
    end
    return ax + s * abx, ay + s * aby           -- a + s * ab
end
~~~

### Point on ray

Also here we just need to test whether the point is not before the origin of the ray.

~~~ Lua
function pointOnRay(ax, ay, bx, by, cx, cy)
    local abx, aby = bx - ax, by - ay        -- ab
    local acx, acy = cx - ax, cy - ay        -- ac
    local t = cross(abx, aby, acx, acy)      -- ab x ac
    if t > 0.00001 or t < -0.00001 then
        return false
    end
    return dot(abx, aby, acx, acy) >= 0      -- ab . ac
end
~~~

## Subdivision

### Subdivision of a line segment

Dividing a line segment into smaller segments, or finding the middle of a line segment is very straightforward when we use the vector equation. Given a segment from a to b, thus using the equation $$a+s*(b-a)$$, we find the middle at s=0.5. Similarly we can get the points at ⅓rd or ⅔rd or any other possible fraction. It's a simple linear interpolation between two points, since that is what the vector equation of a line defines.

Note that if specifically the middle of the line is needed, the equation can be simplified as 

$$
a+0.5*(b-a)=a+0.5*b-0.5a=0.5*a+0.5*b=0.5*(a+b)
$$

So we only need to add the points and divide the result by 2.

### Cutting a line segment interactively

Splitting a line in two lines is easy enough, but how do we know where the user wants it cut? There are several solutions to this.

We can use vector projection to find the closest line as well as the point on that line closest to where the user clicked. The squared distance to the line can be the deciding factor whether to cut or not, whether the user clicked near enough.

Alternatively we can let the user do a cut gesture, which represents a line segment. Now we can check which line segments are being intersected by the cut segment as well as calculate the intersection points where the cut was made.

## A note on axis aligned lines

When lines are axis aligned, quite some simplifications can be done.

### Horizontal lines

Remember that the cross product is defined as $$a_x*b_y-a_y*b_x$$. For a horizontal line we can choose $$c$$ and $$d$$ so that $\vec{cd}$ is equal to $$\langle -1,0\rangle$$ which makes $$\vec{ab}\times\vec{cd}=ab_y$$.

Similarly, $$\vec{ac}\times\vec{cd}$$ becomes $$ac_y$$. So we can write a specific function for horizontal lines as follows.

```lua
function intersectLineSegmentHorizontalLine(ax, ay, bx, by, y)
    local abx, aby = bx - ax, by - ay           -- ab
                                                -- cd is (-1,0)
    local s = aby                               -- ab x cd
    if s < 0.00001 and s > -0.00001 then
        -- Collinear if ac x cd == 0, parallel otherwise
        return nil
    end
    s = acy / s                                 -- ac x cd / ab x cd
    if s < 0 or s > 1 then
        return nil
    end
    return ax + s * abx, ay + s * aby           -- a + s * ab
end
```

#### Vertical lines

For vertical lines we can choose our vector as $$\langle 0,1\rangle$$ which gives us a similar solution.

```lua
function intersectLineSegmentVerticalLine(ax, ay, bx, by, x)
    local abx, aby = bx - ax, by - ay           -- ab
                                                -- cd is (0,1)
    local s = abx                               -- ab x cd
    if s < 0.00001 and s > -0.00001 then
        -- Collinear if ac x cd == 0, parallel otherwise
        return nil
    end
    s = acx / s                                 -- ac x cd / ab x cd
    if s < 0 or s > 1 then
        return nil
    end
    return ax + s * abx, ay + s * aby           -- a + s * ab
end
```

#### 