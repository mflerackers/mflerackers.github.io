---
layout: default
title: Points and Vectors
math: true
---

[Next - Angles](2-angles.html)

# [Math for game development](../) : Points and Vectors

## Scalars
A scalar has one dimension, so we could as well have called it a number.
## Points
A 2 dimensional point, defined as (x, y) describes a position or coordinate in 2D. Points can define the position of an object or camera. They can define a path or way-points which a character needs to follow. Or they can define the shape of an object as for example the points of a polygon.
A point doesn't have to define a global position on the screen. In the case of a moving polygon, the points can define the polygon in a local coordinate system. These points are then transformed to the global coordinate system before being transformed to the screen to be drawn.
### Scaling from the origin
At this stage we can't move or rotate yet, but we can scale. Scaling a point or set of points is done by multiplying each dimension with a scalar.

$$(x',y') = s(x, y)=(s*x, s*y)$$

{% highlight lua %}
function mul(s, x, y)
    return x * s, y *s
end
{% endhighlight %}

It should be noted however that the origin of the scale is (0, 0). This means that everything scales with respect of the origin. This is logical of course, as the origin is the only point that isn't affected 

$$s(0,0)=(0,0)$$ 

All the other points will move away from the origin when the scale factor is greater than 1 or move towards the origin when the scale factor is smaller than 1.
## Vectors
When we subtract two points, we get a vector. Vectors are also defined as a pair $$\langle x, y\rangle$$, however the x and y are no longer a position, but rather define a direction and distance from the origin. A vector $$\langle x, y\rangle$$ can be thought of as an arrow starting from $$(0, 0)$$ towards $$(x, y)$$. The further a vector is from the origin, the greater its magnitude.
If we have two points a and b, we can build the vector $\vec{ab}$ by subtracting a from b.

$$\vec{ab}=b-a$$

{% highlight lua %}
function sub(x1, y1, x2, y2)
    return x1 - x2, y1 - y2
end
{% endhighlight %}

**The point at the arrow of the vector should always come first.**
Notice that calculating the vector from for example $$(0, 0)$$ to $$(1, 4)$$ or from $$(4, 5)$$ to $$(5, 9)$$ both give the vector $$\langle 1, 4\rangle$$. Because to move from any of the two origins towards their destination you need to move in the same direction, crossing the same distance.

Adding the vector to its origin gives us the destination.

$$a + \vec{ab}=b$$

This is quite logical

$$a + \vec{ab} = a + (b - a)=a + b - a=b$$

A vector can move or translate a point by a given distance in a given direction.

{% highlight lua %}
function add(x1, y1, x2, y2)
    return x1 + x2, y1 + y2
end
{% endhighlight %}

Since a point and a vector are so similar, should we really distinguish them? Especially since any point can also be seen as a vector starting from the origin

$$(x, y) - (0, 0) = \langle x, y\rangle$$

Yes and no. In code, we almost never really distinguish the two. In our functional lua code, we use a plain pair of floats x and y for both, and in an object oriented framework we would just use a point or vector class, not both.

However there is an important difference between them. A point defines a coordinate, a vector defines a direction and distance. When we transform points and vectors to a different coordinate system, points are translated, rotated and scaled. Vectors however are only ever rotated and scaled, never translated.

While a point gets translated by adding a vector, a vector doesn't get translated by adding another vector. Vector addition changes the direction and magnitude of the vector.

$$\langle x_3, y_3\rangle=\langle x_1,y_1\rangle+\langle x_2,y_2\rangle$$

For example if we have the vector $$\langle 1,0\rangle$$ pointing along the positive x axis, and the vector $$\langle 0,1\rangle$$ pointing along the positive y axis, we get when adding them

$$\langle 1, 1\rangle=\langle 1,0\rangle+\langle 0,1\rangle$$

This new vector $$\langle 1,1\rangle$$ is pointing diagonally towards the positive x, y plane, and its magnitude is no longer 1. Since we only translated the world, directions and magnitudes shouldn't change. We will later see how we can enforce not having vectors translated when using matrices.

### Scaling from a given center

Now that we can translate, we can devise a method to scale from any given point, not just from the origin. If we want to scale from a point $$(c_x, c_y)$$, we need to place that point at the origin. We can do this by translating the point by adding the vector $$\langle -c_x, -c_y\rangle$$ or subtracting the vector $$\langle c_x, c_y\rangle$$

$$(x',y')=(x, y) - \langle c_x, c_y\rangle$$

If we scale now, the point $$(c_x, c_y)$$ will be the center of our scale. It will not be affected, and all points will drift away from or towards it

$$(x',y')=s((x, y) - \langle c_x, c_y\rangle)$$

However, everything will also be translated, which is not what we wanted, so we need to translate everything back again after scaling

$$(x',y')=s((x, y) - \langle c_x, c_y\rangle) + \langle c_x, c_y\rangle$$

{% highlight lua %}
function scale(x, y, s, cx, cy)
    cx = cx or 0
    cy = cy or 0
    return s*(x-cx)+cx, s*(y-cy)+cy
end
{% endhighlight %}

Besides adding or subtracting vectors, or doing scalar multiplication or division, we can also take the product of two vectors. There are two different kinds of products.

### The dot product

The dot product is defined as the scalar obtained by

$$\langle x_1,y_1\rangle.\langle x_2,y_2\rangle=x_1*x_2+y_1*y_2$$

{% highlight lua %}
function dot(x1, y1, x2, y2)
    return x1*x2+y1*y2
end
{% endhighlight %}

The dot product can be used to calculate the length (magnitude or norm) of a vector. But it can also give you the cosine of the angle between two vectors or project one vector onto another one.
### The cross product
The cross product if defined as

$$\langle x_1,y_1\rangle\times\langle x_2,y_2\rangle=x_1*y_2-y_1*x_2$$

{% highlight lua %}
function cross(x1, y1, x2, y2)
    return x1*y2-y1*x2
end
{% endhighlight %}

In 3 dimensions this product would produce a vector, namely the vector perpendicular to the plane defined by the two vectors. In 2D however, we get a scalar, as there is no third dimension. The cross product can give you the sine of the angle between two vectors.
Combined, the dot and cross products can be used to rotate points.
As these two products have many applications. I would suggest that if you want to memorize anything, memorize these two products, as they can be used in many ways.

### The length of a vector

If we take the dot product of a vector with itself, we obtain its length squared. 

$${\left|\langle x, y\rangle\right|}^2 = \langle x,x\rangle.\langle x,y\rangle=x*x+y*y$$

When we need the actual length, we need to take the square root

$$\left|\langle x, y\rangle\right| = \sqrt{\langle x,x\rangle.\langle x,y\rangle}=\sqrt{x*x+y*y}$$

{% highlight lua %}
function length(x, y)
    return math.sqrt(x*x+y*y)
end
{% endhighlight %}

Being able to obtain the length of a vector allows us to calculate the distance between two points. Given the points a and b, we can create the vector $$\vec{ab}$$ and take it's length. It gives us the distance to travel from a to b.

{% highlight lua %}
function distance(x1, y1, x2, y2)
    local dx, dy = x1-x2, y1-y2
    return math.sqrt(dx*dx+dy*dy)
end
{% endhighlight %}

**Note that when comparing magnitudes, you can often do with the length squared, which saves you a square root calculation.**

if 

$$\sqrt{a} \leq\sqrt{b}$$

then

$$a\leq b$$

If only one side has a square root, we can square the expression, so if 

$$\sqrt{a} \leq b$$

then

$$a\leq b*b$$

A useful application of this is testing whether a point is inside a circle. To know the answer, we need to compare the distance between the point and the center of the circle with the radius of the circle. However instead of using a square root, we can compare the distance squared with the radius squared

{% highlight lua %}
function inCircle(x, y, cx, cy, r)
    local dx, dy = x-cx, y-cy
    return dx*dx+dy*dy <= r*r
end
{% endhighlight %}

Being able to determine the length of a vector, we can see now that for example $$\langle 2, 0\rangle$$ and $$\langle 0, 2\rangle$$ have the same length. We are now able to compare the magnitudes of vectors while disregarding their direction.

$$\sqrt{2*2+0*0} = \sqrt{0*0+2*2}=\sqrt{4}=2$$

### The direction of a vector
While previously we disregarded direction in order to compare lengths, we will now concentrate on the direction dimension of a vector instead. If we erase the magnitude from our vectors, we can determine whether they have the same direction. We can do this by dividing a vector by its length. This process is also called normalization.

$$\frac{\langle x, y\rangle}{\left|\langle x, y\rangle\right|}$$

{% highlight lua %}
function normalize(x, y)
    local l = math.sqrt(x*x+y*y)
    return x/l, y/l
end
{% endhighlight %}

Normalized vectors have always 1 as length, since their original length has been erased. We can now see that $$\langle 1, 3\rangle$$ an $$\langle 2, 6\rangle$$ point in the same direction, as normalization gives for both $$\langle 0.31622776601, 0.94868329805\rangle$$, while $$\langle -1, -3\rangle$$ points in the opposite direction $$\langle -0.31622776601, -0.94868329805\rangle$$.

### Vector negation
Negating or multiplying a vector by -1 gives the vector pointing in the opposite direction.

{% highlight lua %}
function negate(x, y)
    return -x, -y
end
{% endhighlight %}

### Clockwise and counterclockwise perpendicular vector
Sometimes we need the vector perpendicular to a certain vector. There are two possibilities, one in the clockwise direction, and one in the counterclockwise direction.

{% highlight lua %}
function ccw(x, y)
    return -y, x
end

function cw(x, y)
    return y, -x
end
{% endhighlight %}

### A note on notation

A point $$(x, y)$$ is written with round brackets, while I use square brackets to indicate that we're talking about a vector $$\langle x, y\rangle$$. When working with named vectors, I use an arrow $$\overrightarrow{a}$$ or in case of a unit vector a hat $$\hat{a}$$.

[Next - Angles](2-angles.html)