---
title: Angles
math: true
excerpt: "How do we go from vectors to angles and back. What is PI, cosine and sine exactly, and how can they help us to rotate points."
---

| : ---- : |
| [Previous - Points and Vectors](1-points-and-vectors.html) | [Next - Vector projection](3-vector-projection.html) |

## Angles
While the direction of a vector is useful in many ways, we can't tell much from the directions of two vectors, not like we could from their lengths. Nor can we simply linearly interpolate between two directions. For this we need a conversion to something we are more accustomed too, namely angles. Instead of degrees, we are going to use radians. While degrees are handy for display or editing values, converting from and to degrees would waste a lot of unnecessary CPU cycles. Degrees go from 0 to 360, radians from 0 to $$2\pi$$.

~~~ lua
function deg2rad(d)
    return d*math.pi/180
end

function rad2deg(r)
    return r * 180 / math.pi
end
~~~

The number $$\pi$$ used in trigonometric functions is not an arbitrarily chosen value. When we work with normalized vectors, vectors with a length of 1, their destination lies on a circle with a radius of 1. If you measure the circumference of that circle, you arrive at exactly $$2\pi$$ or 6.28318530718. You might remember that if you need the circumference of a circle with radius r, that you need to calculate $$2\pi r$$. The $$r$$ in this case is nothing but a scale factor by which you scale the unit circle which has a radius of 1 and a circumference of $$2\pi$$.

Some game engines use angles from 0 to 360 degrees or from 0 to 1 percent instead. Some even use a clockwise direction for angles rather than the counterclockwise direction used in mathematics to compensate for the inverted y-axis (many games use a positive y-axis going down instead of up). However be aware that this may require changes in mathematical calculations everywhere. The order of vectors in cross products may need to be switched, derivatives may need additional multipliers and angles might need to be adapted.

### Angle to vector
Lets look at some vectors and the angles they represent.

| vector                   | angle              |
| ------------------------ | ------------------ |
| $$\langle 1, 0\rangle$$  | 0 or $$\pi$$       |
| $$\langle 0, 1\rangle$$  | $$\frac{\pi}{2}$$  |
| $$\langle -1, 0\rangle$$ | $$\pi$$            |
| $$\langle 0, -1\rangle$$ | $$\frac{3\pi}{2}$$ |

Unfortunately, there is no easy way to go from an angle to a direction and back. We need trigonometric functions to do this, which are rather slow compared to normal operations. To go from an angle to a direction vector, we use cosine and sine. These give the x and y of the normalized vector representing the direction.

~~~ lua
function vectorFromAngle(a)
    return math.cos(a), math.sin(a)
end
~~~

### Vector to angle
To get the angle represented by a given normalized vector, we might be tempted to use the reverse calculation, namely arc cosine or arc sine. However if we use $$acos(x)$$, we get an angle between $$0$$ and $$\pi$$ radians. This is because without using y, we don't know whether the vector is pointing up or down, it only tells us left or right. Similarly, $$asin(y)$$ gives us an angle between $$-\frac{\pi}{2}$$ and $$\frac{\pi}{2}$$ radians as it can't tell whether the vector is pointing right or left.
Lucky for us, we don't have to do the necessary tests ourselves, as most mathematics modules have an atan2 function taking both x and y to return an angle between $$0$$ and $$\pi$$ radians. The tangent of an angle is the slope of the vector, or how many y units for every x unit. If we express this in angles instead of coordinates, we get

$$tan(\theta)=\frac{sin(\theta)}{cos(\theta)}=\frac{y}{x}$$

Using atan on both sides we get

$$\theta=atan(\frac{sin(\theta)}{cos(\theta)})=atan(\frac{y}{x})$$

If we would use the tan function, we would get the same problem as before, as with only the slope, the result of $$\frac{y}{x}$$, it will return an angle between $$-\frac{\pi}{2}$$ and $$\frac{\pi}{2}$$ radians. The previously mentioned atan2 taking both x and y as input will give us the correct angle.

$$\theta=atan2(sin(\theta), cos(\theta))=atan2(y, x)$$

~~~ lua
function angleFromVector(x, y)
    return math.atan2(y, x)
end
~~~

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

~~~ lua
function angleBetweenVectors(x1, y1, x2, y2)
    return math.atan2(cross(x1, y1, x2, y2), dot(x1, y1, x2, y2))
end
~~~

Note that we can now explain why the length formula is the square root of the dot product. The angle between a vector and itself is 0 so we get

$$cos(0)=\frac{\vec{a}.\vec{a}}{\left|\vec{a}\right|\left|\vec{a}\right|}$$

The cosine of 0 radians is 1

$$1=\frac{\vec{a}.\vec{a}}{\left|\vec{a}\right|^2}$$

Multiplying both sides with $$\left\vert\vec{a}\right\vert^2$$ gives

$$\left|\vec{a}\right|^2=\vec{a}.\vec{a}$$

Taking the square root of both sides gives us that the length of a vector is the square root of the dot product of the vector with itself.

$$\left|\vec{a}\right|=\sqrt{\vec{a}.\vec{a}}$$

### Determining whether 2 vectors point in the same direction
Since we can calculate the angle between two vectors now, you might think of looking at the angle. If the angle between the vectors is between $$-\frac{\pi}{2}$$ and $$\frac{\pi}{2}$$ radians, the vectors are pointing in the same direction. However, we can simplify that. If we calculate just the cosine of the angle we can look whether that cosine is greater than 0. For the cosine we just need the dot product of the vectors and their lengths.

$$cos(\theta)=\frac{\vec{a}.\vec{b}}{\left|\vec{a}\right|\left|\vec{b}\right|}$$

But since we are only interested in the sign, we can disregard the lengths. So if $$\vec{a}.\vec{b} > 0$$ we know that the vectors point in the same direction.
An application of this would be for example to draw or attack enemy units only if they are in front of us.

~~~ lua
function drawEnemy(x, y)
    local toEnemyX, toEnemyY = sub(x, y, ox, oy)
    local cosine = dot(toEnemyY, playerDir)
    if cosine > 0 then
        // actual drawing procedure
    else
        // nothing to do, enemy is behind our back
    end
end
~~~

## Rotation

### Rotation around the origin
We will start by trying to rotate the normalized vector $$\vec{b}$$ by $$\alpha$$ radians. If we have the normalized vector $$\hat{b}$$ pointing in the direction of $$\beta$$ radians, it can be written as $$\hat{b}=(cos(\beta), sin(\beta))$$. Similarly, we can write the angle $$\alpha$$ as the normalized vector $$\hat{a} =(cos(\alpha), sin(\alpha))$$ pointing in the direction of $$\alpha$$. As we saw before, for normalized vectors, the cosine of the angle between two vectors was equal to the dot product of the two vectors, while the sine of the angle between two vectors was equal to the cross product. The angle between the vectors $$\hat{a}$$ and $$\hat{b}$$ is equal to $$\alpha-\beta$$, so we can write

$$cos(\alpha-\beta)=cos(\beta)*cos(\alpha)+sin(\beta)*sin(\alpha)$$

$$sin(\alpha-\beta)=cos(\beta)*sin(\alpha)-sin(\beta)*cos(\alpha)$$

Since multiplication is commutative we can write this as

$$cos(\alpha-\beta)=cos(\alpha)*cos(\beta)+sin(\alpha)*sin(\beta)$$

$$sin(\alpha-\beta)=sin(\alpha)*cos(\beta)-cos(\alpha)*sin(\beta)$$

What we are looking for, in order to rotate, are the cosine and sine of $$\alpha+\beta$$. So we need to replace $$\beta$$ with $$-\beta$$

$$cos(\alpha+\beta)=cos(\alpha)*cos(-\beta)+sin(\alpha)*sin(-\beta)$$

$$sin(\alpha+\beta)=sin(\alpha)*cos(-\beta)-cos(\alpha)*sin(-\beta)$$

The cosine can discard the minus, as $$cos(-\beta)=cos(\beta)$$, the sine has to switch signs, as $$sin(-\beta)=-sin(\beta)$$. So we arrive at

$$cos(\alpha+\beta)=cos(\alpha)*cos(\beta)-sin(\alpha)*sin(\beta)$$

$$sin(\alpha+\beta)=sin(\alpha)*cos(\beta)+cos(\alpha)*sin(\beta)$$

These formula's might look familiar, as they should be featured in every mathematics book on trigonometry. They can be easily derived from the dot and cross products as shown above.

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

### Rotation around a given center
Like scaling from a given center, the center of rotation can be selected by translating the desired center to the origin, doing the rotation and translating back.

$$x'=(x-c_x)*cos(\alpha)-(y-c_y)*sin(\alpha) + cx$$

$$y'=(x-c_x)*sin(\alpha)+(y-c_y)*cos(\alpha) + cy$$

Of course unless this rotation anchor is animated, it is better to offset the geometry once in order to save on calculations. For example instead of drawing a sprite as {0, 0, w, h}, it is better drawn as {-w*0.5, -h*0.5, w*0.5, h*0.5} if the anchor is always in the center of the sprite. The same goes for scaling. If a sprite has a fixed scale, incorporate it into the geometry, like {0, 0, w*2, h*2}.

~~~ lua
function rotate(x, y, angle, cx, cy)
    cx = cx or 0
    cy = cy or 0
    local c = math.cos(angle)
    local s = math.sin(angle)
    x = x - cx
    y = y - cy
    return x*c - y*s + cx, x*s + y*c + cy
end
~~~

### Rotation using a normalized vector

Note that while we seemingly use an angle to rotate, this is not really the case. Rotation is actually done using a vector. If we look at the formula for rotating around $$(0, 0)$$

$$x'=x*cos(\alpha)-y*sin(\alpha)$$

$$y'=x*sin(\alpha)+y*cos(\alpha)$$

We can replace the sine and cosine of $$\alpha$$ with the normalized vector $$\hat{v}$$

$$\hat{v}=\langle cos(\alpha), sin(\alpha)\rangle$$

giving

$$x'=x*v_x-y*v_y$$

$$y'=x*v_y+y*v_x$$

This insight has several implications. If we want to rotate for example a sprite given a direction, we don't need to calculate an angle from the direction, to convert it again to a vector to do the rotation. We can just use the normalized vector directly. If we apply the rotation of a static object every frame, we might want to store this vector rather than the angle, since we can use it directly without the need of trigonometric functions.

~~~ lua
function followTarget(x, y, tx, ty, points)
    -- Build direction vector
    local vx, vy = tx-x, ty-y
    -- Get angle
    local angle = math.atan2(vy, vx)
    -- Get vector
    local c = math.cos(angle)
    local s = math.sin(angle)
    -- Rotate points
    for _, point in ipairs(points) do
        point.x, point.y = point.x*c - point.y*s, point.x*s + point.y*c
    end
end
~~~

compare this with

~~~ lua
function followTarget(x, y, tx, ty, points)
    -- Build direction vector
    local vx, vy = normalize(tx-x, ty-y)
    -- Rotate points
    for _, point in ipairs(points) do
        point.x, point.y = point.x*vx - point.y*vy, point.x*vy + point.y*vx
    end
end
~~~

| : ---- : |
| [Previous - Points and Vectors](1-points-and-vectors.html) | [Next - Vector projection](3-vector-projection.html) |
