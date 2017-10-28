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

$$\vec{ab}=b-a$$

Our line can now be defined using the point $$a$$ and the vector $$\vec{ab}$$.

$$p=a+s*\vec{ab}$$

For every scalar s the point p will be a point on our line.

### Line intersection

Given the lines 

If these lines intersect, then there is a point p for which there is both a scalar t and a scalar s for which the following equations are equal to p.

$$p=a+s*\vec{ab}$$

$$p=c+t*\vec{cd}$$

Since p is the same in this case we can write

$$a+s*\vec{ab}=c+t*\vec{cd}$$

If we subtract c from both sides we get

$$a-c+s*\vec{ab}=t*\vec{cd}$$

The point $$c$$ subtracted from $$a$$ gives us the vector $$\vec{ca}$$

$$\vec{ca}+s*\vec{ab}=t*\vec{cd}$$

Now we are going to use the cross product with $$\vec{ab}$$ to get rid of this term, as $$\vec{ab}\times\vec{ab}$$ is 0.

$$\vec{ca}\times\vec{ab}+s*\vec{ab}\times\vec{ab}=t*\vec{cd}\times\vec{ab}$$

Erasing the 0 term gives us

$$\vec{ca}\times\vec{ab}=t*\vec{cd}\times\vec{ab}$$

This means t is

$$\frac{\vec{ca}\times\vec{ab}}{\vec{cd}\times\vec{ab}}=t$$

Before we go on, let's note that $$\vec{cd}\times\vec{ab}$$ can be 0, and we shouldn't divide by 0. This cross product is zero when our lines are parallel and thus have no intersection. Remember, the cross product gives us the sine of the angle (multiplied with a factor if the vectors are not normalized). The sine is 0 in the cases of 0 and 180 degrees. If there's an angle of 0 degrees between the lines, they are definitely parallel.

Now that we know t, we can calculate the point p from $$p=c+t*\vec{cd}$$. This gives us the intersection point.

{% highlight lua %}
function intersectLines(x1, y1, x2, y2, x3, y3, x4, y4)
    x2, y2 = x2 - x1, y2 - y1               -- ab
    x4, y4 = x4 - x3, y4 - y3               -- cd
    local t = cross(x4, y4, x2, y2)         -- cd x ab
    if t < 0.00001 and t > -0.00001 then
        return nil
    end
    t = cross(x1 - x3, y1 - y3, x2, y2) / t -- ca x ab / cd x ab
    return x3 + t * x4, y3 + t * y4         -- c + t * cd
end
{% endhighlight %}

### Point on line

Let's say we have the line through the points a and b, and we need to know whether c is on the same line. Remember how we tested for parallel lines using the cross product? That is exactly what we need now. If we build a vector from a to c called ac, and cross it with the vector ab, the result is 0 if c lies on ab.

{% highlight lua %}
function pointOnLine(x1, y1, x2, y2, x3, y3)
    x2, y2 = x2 - x1, y2 - y1 -- ab
    x3, y3 = x3 - x1, y3 - y1 -- ac
    local t = cross(x2, y2, x3, y3)
    return t < 0.00001 and t > -0.00001
end
{% endhighlight %}

## Line segments

Sometimes our lines don't go to infinity in both directions, but have given boundaries. Line segments are not much different in usage, except for some extra testing to make sure we are actually still on the line segment. Given a segment between points $$a$$ and $$b$$, we saw we could define the line as

$$p=a+s*\vec{ab}$$

We can see that if s is 0, p is equal to a, while if s is 1, p is equal to b. Any value of s not between 0 and 1 gives a point outside of the line segment.

### Line segment intersection

Given what we know from line intersection, intersecting a line or line segment with another line is not so hard. We only need to be sure that the intersection point is actually within the segment(s). This is simple if we have s for the point on the segment. If s is outside the interval [0,1], the point is outside the range of the segment.

{% highlight lua %}
function intersectLineLineSegment(x1, y1, x2, y2, x3, y3, x4, y4)
    x2, y2 = x2 - x1, y2 - y1               -- ab
    x4, y4 = x4 - x3, y4 - y3               -- cd
    local t = cross(x4, y4, x2, y2)         -- cd x ab
    if t < 0.00001 and t > -0.00001 then
        return nil
    end
    t = cross(x1 - x3, y1 - y3, x2, y2) / t -- ca x ab / cd x ab
    if t < 0 or t > 1 then
        return nil
    end
    return x3 + t * x4, y3 + t * y4         -- c + t * cd
end
{% endhighlight %}

### Point on line segment

We first need to test whether the point lies on the line using the cross product as we did before for infinite lines.

If the point is on the line, we need to test whether it lies between the segment's boundaries. We can do this second step partly by calculating the dot product, like when we did vector projection. If the dot product is smaller than 0, the point lies before the start of the segment, as the angle is greater than 90 degrees in that case.

To be sure that the point doesn't lie past the end of the segment, we can look whether the dot product of $$\vec{ac}$$ with itself is smaller than the dot product of $$\vec{ab}$$ with itself, as the dot product gives us the length squared.

{% highlight lua %}
function pointOnLineSegment(x1, y1, x2, y2, x3, y3)
    x2, y2 = x2 - x1, y2 - y1               -- ab
    x3, y3 = x3 - x1, y3 - y1               -- ac
    local t = cross(x2, y2, x3, y3)         -- ab x ac
    if t > 0.00001 or t < -0.00001 then
        return false
    end
    t = dot(x2, y2, x3, y3)                 -- ab . ac
    if t < 0 then
        return false
    end
    t = dot(x3, y3, x3, y3)                 -- ac . ac
    return t <= dot(x2, y2, x2, y2)         -- ab . ab
end
{% endhighlight %}