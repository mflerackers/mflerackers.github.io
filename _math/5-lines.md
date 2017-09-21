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

Before we go on, let's note that $$\vec{cd}\times\vec{ab}$$ can be 0, and we shouldn't divide by 0. This cross product is zero when our lines are parallel and thus have no intersection. Remember, the cross product gives us the sine of the angle (multiplied with a factor if the vectors are not normalized). The sine is 0 in the cases of 0 and 180 degrees. If there's and angle of 0 degrees between the lines, they are definitely parallel.

Now that we know t, we can calculate the point p from $$p=c+t*\vec{cd}$$. This gives us the intersection point.

{% highlight lua %}
function intersectLines(x1, y1, x2, y2, x3, y3, x4, y4)
    x2, y2 = x2 - x1, y2 - y1
    x4, y4 = x4 - x3, y4 - y3
    local divisor = cross(x2, y2, x4, y4)
    if cross < 0.00001 then return nil
    return cross(x3 - x1, y3 - y1) / divisor
end
{% endhighlight %}

### Line segments

Sometimes our lines don't go to infinity in both directions, but have given boundaries. Line segments are not much different in usage, except for some extra testing to make sure we are actually still on the line segment. Given a segment between points $$a$$ and $$b$$, we get 

$$p=a+s*\vec{ab}$$