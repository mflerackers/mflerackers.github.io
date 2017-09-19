---
layout: default
title: Vector projection
math: true
---

[Previous - Points and Vectors](2-angles.html)

# [Math for game development](../) : Vector projection

## Vector projection

A projection of a point or vector onto a line or other vector has many useful applications.

Imagine for example a cart on a slope, given two points on the slope and the gravity vector, we can use the projection of the gravity vector onto the slope to calculate the direction of the force on the cart at that point.

Or imagine for example a character jumping along an irregular polygon wall, when its collision circle collides with the wall, where is the closest point to the wall? This point is the projection of the circle's center onto the wall.

## Calculating the projection of a vector onto another vector

If we look at the picture below, we see $$A'$$ which is the projection of the vector $$\overrightarrow{CA}$$ or $$\hat{a}$$ onto $$\overrightarrow{CB}$$ or $$\hat{b}$$. The goal is to calculate $$A'$$ in terms of $$\hat{a}$$ and $$\hat{b}$$.

![proj_unrotated](/assets/proj_unrotated.png)

To make this task easier, or at least more intuitive, we are going to rotate this setup. Let's arrange the vector $$b$$ so that it is parallel with the x-axis.

![proj_unscaled_equal](/assets/proj_unscaled_equal.png)

If we look at point $$A'$$ now, we see that $$A'=(x, 0)$$ if $$A=(x,y)$$. Now let's scale our setup, so that $$A$$ is lying on the unit circle, a circle with as radius 1. In this case $$x$$ is equal to the cosine of $$\alpha$$.

![proj_cos](/assets/proj_cos.png)

$$x=cos(\alpha)$$

Of course in our second rotated but unscaled scenario, the vector $$\overrightarrow{CA}$$ doesn't necessarily have a length equal to 1. Both $$A$$ and $$A'$$ are scaled by the length of $$\overrightarrow{CA}$$. This means that to go back to the second scenario, have to multiply by the length of $$\overrightarrow{CA}$$ to correct the scale.

$$x=\lvert\overrightarrow{CA}\lvert cos(\alpha)=\lvert a\lvert cos(\alpha)\quad(1)$$

We don't want $$\alpha$$ in our equation, as we want a solution with just the two vectors $$a$$ and $$b$$. We can use the dot product, since the cosine of an angle is equal to the dot product of the vectors that form the angle if the vectors are normalized

$$cos(\alpha)=\hat{a}.\hat{b}$$

A normalized vector is one whose length is 1, thus it is obtained by dividing a vector by its length. Doing this with vectors a and b gives

$$cos(\alpha)=\frac{a.b}{\lvert a\lvert\lvert b\lvert}\quad(2)$$

Substituting the cosine in (1) by (2) gives

$$x=\lvert a\lvert\frac{a.b}{\lvert a\lvert\lvert b\lvert}$$

Or simplified, since we can ignore multiplying and dividing by the same factor $$\lvert a\lvert$$

$$x=\frac{a.b}{\lvert b\lvert}$$

Our projection of point $$A$$ in scenario 2 is thus point $$A'=(x,0)$$ or $$A'=x*(1,0)$$. 

$$A'=\frac{a.b}{\lvert b\lvert}(1, 0)$$

We chose $$\overrightarrow{CB}$$ to be parallel with the x-axis to clearly see that x was equal to the scaled cosine. But in our original scenario $$\overrightarrow{CB}$$ is not parallel to the x-axis. the only difference with scenario one and two is that we rotated everything. This rotation kept the sizes of the vectors as well as the angle between them intact. The only difference that affects the outcome is that $$b$$ is rotated. As we can see,the projected point $$A'$$ always lies somewhere along $$b$$. In scenario two, $$A'$$ only lies along the x-axis because b is parallel with it. The vector $$(1,0)$$ is nothing more than the normalized vector $$\hat{b}$$. Thus our projection of a on b in scenario one is

$$proj(a,b)=\frac{a.b}{\lvert b\lvert}\hat{b}$$

Since $$\hat{b}$$ is equal to $$b$$ divided by it's length $$\lvert b\lvert$$ we can write

$$proj(a,b)=\frac{a.b}{ {\lvert b\lvert}^2}b$$

And since the length $$\lvert b\lvert$$ is the square root of the dot product of b with itself or $$\lvert b\lvert=\sqrt{b.b}$$ we get

$$proj(a,b)=\frac{a.b}{b.b}b$$

So the projection of $$a$$ on $$b$$ is the dot product of $$a$$ and $$b$$ divided by the dot product of $$b$$ with itself multiplied with b.

