---
math: true
title: Angles
---

# Math for game development : Angles

## Angles
While the direction of a vector is useful in many ways, we can't tell much from the directions of two vectors, not like we could from their lengths. Nor can we simply linearly interpolate between two directions. For this we need a conversion to something we are more accustomed too, namely angles. Instead of degrees, we are going to use radians. While degrees are handy for display or editing values, converting from and to degrees would waste a lot of unnecessary CPU cycles. Degrees go from 0 to 360, radians from 0 to $$2\pi$$.

{% highlight lua %}
function deg2rad(d)
    return d*math.pi/180
end

function rad2deg(r)
    return r * 180 / math.pi
end
{% endhighlight %}

## Angle to vector
Lets look at some vectors and the angles they represent.

| vector  | angle            |
| ------- | ---------------- |
| (1, 0)  | 0 or $$\pi$$       |
| (0, 1)  | $$\frac{\pi}{2}$$  |
| (-1, 0) | $$\pi$$            |
| (0, -1) | $$\frac{3\pi}{2}$$ |

Unfortunately, there is no easy way to go from an angle to a direction and back. We need trigonometric functions to do this, which are rather slow compared to normal operations. To go from an angle to a direction vector, we use cosine and sine. These give the x and y of the normalized vector representing the direction.

{% highlight lua %}
function vectorFromAngle(a)
    return math.cos(a), math.sin(a)
end
{% endhighlight %}

### Vector to angle
To get the angle represented by a given normalized vector, we might be tempted to use the reverse calculation, namely arc cosine or arc sine. However if we use $$acos(x)$$, we get an angle between $$0$$ and $$\pi$$ radians. This is because without using y, we don't know whether the vector is pointing up or down, it only tells us left or right. Similarly, $$asin(y)$$ gives us an angle between $$-\frac{\pi}{2}$$ and $$\frac{\pi}{2}$$ radians as it can't tell whether the vector is pointing right or left.
Lucky for us, we don't have to do the necessary tests ourselves, as most mathematics modules have an atan2 function taking both x and y to return an angle between $$0$$ and $$\pi$$ radians. The tangent of an angle is the slope of the vector, or how many y units for every x unit. If we express this in angles instead of coordinates, we get

$$tan(\theta)=\frac{sin(\theta)}{cos(\theta)}=\frac{y}{x}$$

Using atan on both sides we get

$$\theta=atan(\frac{sin(\theta)}{cos(\theta)})=atan(\frac{y}{x})$$

If we would use the tan function, we would get the same problem as before, as with only the slope, the result of $$\frac{y}{x}$$, it will return an angle between $$-\frac{\pi}{2}$$ and $$\frac{\pi}{2}$$ radians. The previously mentioned atan2 taking both x and y as input will give us the correct angle.

$$\theta=atan2(sin(\theta), cos(\theta))=atan2(y, x)$$

{% highlight lua %}
function angleFromVector(x, y)
    return math.atan2(y, x)
end
{% endhighlight %}

### Angle between 2 vectors
We could convert both vectors separately to angles and subtract these, but we would be doing more work than is needed. If only we could obtain the sine and cosine of the angle, we could, like before, use the arc tangent to get the angle. This is where the dot and cross products come in handy, as they have the following properties

$$sin(\theta)=\frac{\vec{a}\times\vec{b}}{\left|\vec{a}\right|\left|\vec{b}\right|}$$

and

$$cos(\theta)=\frac{\vec{a}.\vec{b}}{\left|\vec{a}\right|\left|\vec{b}\right|}$$

The cross product divided by the product of the length of the vectors gives us the sine of the angle, while the dot product does the same for the cosine.
Since we actually need the slope, thus sine divided by cosine, we can discard the divisors and obtain

$$\frac{sin(\theta)}{cos(\theta)}=\frac{\vec{a}\times\vec{b}}{\vec{a}.\vec{b}}$$

This means we can use the cross and dot products of the two vectors together with atan2 to obtain $$\theta$$

$$\theta=atan2(\vec{a}\times\vec{b}, \vec{a}.\vec{b})$$

{% highlight lua %}
function angleBetweenVectors(x1, y1, x2, y2)
    return math.atan2(cross(x1, y1, x2, y2), dot(x1, y1, x2, y2))
end
{% endhighlight %}

Note that we can now explain why the length formula is the square root of the dot product. The angle between a vector and itself is 0 so we get

$$cos(0)=\frac{\vec{a}.\vec{a}}{\left|\vec{a}\right|\left|\vec{a}\right|}$$

The cosine of 0 radians is 1

$$1=\frac{\vec{a}.\vec{a}}{\left|\vec{a}\right|^2}$$

Multiplying both sides with 

$$\left|\vec{a}\right|^2$$ 

gives

$$\left|\vec{a}\right|^2=\vec{a}.\vec{a}$$

Taking the square root of both sides gives us that the length of a vector is the square root of the dot product of the vector with itself.

$$\left|\vec{a}\right|=\sqrt{\vec{a}.\vec{a}}$$

### Determining whether 2 vectors point in the same direction
Since we can calculate the angle between two vectors now, you might think of looking at the angle. If the angle between the vectors is between $$-\frac{\pi}{2}$$ and $$\frac{\pi}{2}$$ radians, the vectors are pointing in the same direction. However, we can simplify that. If we calculate just the cosine of the angle we can look whether that cosine is greater than 0. For the cosine we just need the dot product of the vectors and their lengths.

$$cos(\theta)=\frac{\vec{a}.\vec{b}}{\left|\vec{a}\right|\left|\vec{b}\right|}$$

But since we are only interested in the sign, we can disregard the lengths. So if $$\vec{a}.\vec{b} > 0$$ we know that the vectors point in the same direction.
An application of this would be for example to draw or attack enemy units only if they are in front of us.

{% highlight lua %}
function drawEnemy(x, y)
    local toEnemyX, toEnemyY = sub(x, y, ox, oy)
    local cosine = dot(toEnemyY, playerDir)
    if cosine > 0 then
        // actual drawing procedure
    else
        // nothing to do, enemy is behind our back
    end
end
{% endhighlight %}

### Rotating around the origin
We want to rotate the normalized vector $$\vec{a}$$ by $$\beta$$. If we have the normalized vector $$\vec{a}$$, it can be defined as $$(cos(\alpha), sin(\alpha))$$. Similarly, we can make $$\beta$$ into a normalized vector $$\vec{b} =(cos(\beta), sin(\beta))$$. As we saw before, for normalized vectors, the cosine of the angle between two vectors was equal to the dot product of the two vectors, while the sine of the angle between two vectors was equal to the cross product. The angle between the vectors here is equal to $$\alpha-\beta$$, so we get

$$cos(\alpha-\beta)=cos(\alpha)*cos(\beta)+sin(\alpha)*sin(\beta)$$

$$sin(\alpha-\beta)=sin(\alpha)*cos(\beta)-cos(\alpha)*sin(\beta)$$

What we are looking for, in order to rotate, are the cosine and sine of $$\alpha+\beta$$. So we need to replace $$\beta$$ with $$-\beta$$

$$cos(\alpha+\beta)=cos(\alpha)*cos(-\beta)+sin(\alpha)*sin(-\beta)$$

$$sin(\alpha+\beta)=sin(\alpha)*cos(-\beta)-cos(\alpha)*sin(-\beta)$$

The cosine can discard the minus, as $$cos(-\beta)=cos(\beta)$$, the sine has to switch signs, as $$sin(-\beta)=-sin(\beta)$$. So we arrive at

$$cos(\alpha+\beta)=cos(\alpha)*cos(\beta)-sin(\alpha)*sin(\beta)$$

$$sin(\alpha+\beta)=sin(\alpha)*cos(\beta)+cos(\alpha)*sin(\beta)$$

These formula's might look familiar, as they should be featured in every mathematics book on geometry. They can be easily derived from the dot and cross products as shown above.

We rotated the normalized vector pointing towards $$\beta$$ by $$\alpha$$ degrees. We would like to do that for any vector, not just normalized vectors, so that we can apply it to points. This step is easy, as we can multiply both sides by the length of the vector

$$l*cos(\alpha+\beta)=l*cos(\alpha)*cos(\beta)-l*sin(\alpha)*sin(\beta)$$

$$l*sin(\alpha+\beta)=l*sin(\alpha)*cos(\beta)+l*cos(\alpha)*sin(\beta)$$

Since $$(l*cos(\beta), l*sin(\beta))$$ is our point (x, y) to be rotated, we can write

$$l*cos(\alpha+\beta)=x*cos(\alpha)-y*sin(\alpha)$$

$$l*sin(\alpha+\beta)=x*sin(\alpha)+y*cos(\alpha)$$

Similarly $$(l*cos(\alpha+\beta), l*sin(\alpha+\beta))$$ is the rotated point (x', y'). Note that as a vector, the length has been preserved by the rotation.

$$x'=x*cos(\alpha)-y*sin(\alpha)$$

$$y'=x*sin(\alpha)+y*cos(\alpha)$$

This formula might also look familiar if you've played with rotation matrices before.

### Rotating around a given center
Like scaling from a given center, the center of rotation can be selected by translating the desired center to the origin, doing the rotation and translating back.

$$x'=(x-c_x)*cos(\alpha)-(y-c_y)*sin(\alpha) + cx$$

$$y'=(x-c_x)*sin(\alpha)+(y-c_y)*cos(\alpha) + cyl$$

Of course unless this rotation anchor is animated, it is better to offset the geometry once in order to save on calculations. For example instead of drawing a sprite as {0, 0, w, h}, it is better drawn as {-w*0.5, -h*0.5, w*0.5, h*0.5} if the anchor is always in the center of the sprite. The same goes for scaling. If a sprite has a fixed scale, incorporate it into the geometry, like {0, 0, w*2, h*2}.

