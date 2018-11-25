---
title: Interpolation
math: true
excerpt: "What is the winding order of a polygon. How to determine the area. How to know whether a points lies in a polygon. How to determine whether a polygon is convex or concave. How to perform collision detection and response between polygons."
---

| : ---- : |
| [Previous - Polygons](7-polygons.html) | Next - Differentiation |

# Interpolation (Procedural animation)

## Linear interpolation

We will start with a simple linear interpolation between two values. Let's say we have a spaceship which needs to rendezvous with a bigger ship to trade with. The bigger ship is stationery relatively to our ship, so we can just trace a straight line from our current position a to our destination b.

Our function defining our position perfectly at time t is a vector equation for a line

$$
p=t*v+a
$$

With b our start position and v the vector pointing from b to c, our end position, thus

$$
v=b-a
$$

<p data-height="391" data-theme-id="0" data-slug-hash="zMRMJQ" data-default-tab="result" data-user="mflerackers" data-pen-title="Straight line" class="codepen">See the Pen <a href="https://codepen.io/mflerackers/pen/zMRMJQ/">Straight line</a> by Marc (<a href="https://codepen.io/mflerackers">@mflerackers</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

However, what if the docking bay(s) are positioned so that following a line would fly us through the bigger ship? A constraint set by the designers tells us not to fly over or under objects, as they want it to be a 2D game rather than 3D.

Instead of two points, we'll use three in our path. First we find a new point b from which we would approach the ship's docking bay, far enough so we don't intersect the ship. We first fly from a to this waypoint b, and from there to the docking bay at point c.

<p data-height="369" data-theme-id="0" data-slug-hash="eQVbLp" data-default-tab="result" data-user="mflerackers" data-pen-title="Straight lines with waypoint" class="codepen">See the Pen <a href="https://codepen.io/mflerackers/pen/eQVbLp/">Straight lines with waypoint</a> by Marc (<a href="https://codepen.io/mflerackers">@mflerackers</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

While this works, the artists are not going to like this solution. It's not realistic for a ship to fly sharp angles as inertia won't allow for it. The ship would have to slow down on approach of the waypoint, turn, and accelerate again, which is a waste of fuel.

## Curves

what we need is an interpolation between these two linear motions. Note that (if we disregard the scale and offset of t within the inner lerps) what we did until now was to use a step function to select between two functions.

```lua
function step(a, x)
  return (x >= a) and 1 or 0;
end

function lerp(a, b, w)
    return a + (b-a)*w;
end

function trajectory(a, b, c, t)
    return lerp(lerp(a, b, t * 2), lerp(b, c, (t-0.5)*2), step(0.5, t))
end
```

We need something better.

Let's imagine for a moment that the trajectory from b to c is taken by another ship who's destination is also the docking port. We'll use as our path no longer a to b, but a to p, the position of the other ship at that moment in time. Our former trajectory was a line. The new trajectory of our ship is still a line at each point in time, however the destination for the line is now different at each moment in time. Our trajectory across time is no longer a line.

<p data-height="360" data-theme-id="0" data-slug-hash="RqMByW" data-default-tab="result" data-user="mflerackers" data-pen-title="Straight lines with moving waypoint" class="codepen">See the Pen <a href="https://codepen.io/mflerackers/pen/RqMByW/">Straight lines with moving waypoint</a> by Marc (<a href="https://codepen.io/mflerackers">@mflerackers</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

```lua
function trajectory(a, b, c, t)
    local c2 = lerp(b, c, t)
    return lerp(a, c2, t)
end
```

The function our ship is following is dependent on the result of the function of the other ship.

$$
c'=t*(c-b)+b \\
p=t*(c'-a)+a
$$

Filling in c' into the equation for p gives us

$$
p=t*((t*(c-b)+b)-a)+a
$$

writing out the products gives

$$
p=t^2*c-t^2*b+t*b-t*a+a
$$

grouping the terms gives

$$
p=(1-t)a+(t-t^2)b+(t^2)c
$$

Our path is a second order polynomial which goes from a to c and is pulled towards b. What we did is create a spline or curve motion.

## Quadratic bézier curve

Now if we look carefully, we see that one half of the spline is pulled more towards the control point b. This is because we didn't make our start flexible, only our endpoint.

If we make our start point go along a line from a to b. We get

$$
a'=t*(b-a)+a \\
c'=t*(c-b)+b \\
p=t*(c'-a')+a'
$$

Writing out the products and grouping the terms gives us

$$
p=(1-2t+t^2)a+(2t-t^2)b+(t^2)c
$$

<p data-height="364" data-theme-id="0" data-slug-hash="ZmrZzZ" data-default-tab="result" data-user="mflerackers" data-pen-title="Quadratic bezier docking" class="codepen">See the Pen <a href="https://codepen.io/mflerackers/pen/ZmrZzZ/">Quadratic bezier docking</a> by Marc (<a href="https://codepen.io/mflerackers">@mflerackers</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

This is a quadratic Bézier curve. Not the one with the handles in case you are wondering, that is a cubic Bézier curve which is a third order polynomial, while our curve is only second degree.

But as you can see, there is no hard math behind it, it is merely a linear interpolation on top of two linear interpolations.

The grouped terms we obtained are the basis functions of our spline.

$$
1-2t+t^2 \\
2t-2t^2 \\
t^2
$$

They are usually written in the following equivalent form

$$
(1-t)^2 \\
2(1-t)t \\
t^2
$$

We can now make a curved path between two points using one control point. Do we need more? It depends on what we use it for. This path only has one bend in it, since it is a second degree polynomial. What if we need more than one bend, like for zigzagging between two obstacles?

## Cubic bezier curve

A third order Bézier, the one with the handles, has two bends. How to construct those? We have 4 points this time, a start and end point as well as two control points. What we do is another linear blend, this time using points calculated from two quadratic bézier curves instead of two lines. The points for our bézier are a, b, c, d. We start by interpolating on three lines ab, bc and cd. The resulting points are $$a'$$, $$c'$$ and $$d'$$. Then we interpolate on two lines $$a'c'$$ and $$c'd'$$. The resulting points from these, $$a''$$ and $$d''$$, follow quadratic bézier curves. If we do another interpolation on the line between these, we get a cubic bézier curve.

<p data-height="544" data-theme-id="0" data-slug-hash="RYjYgX" data-default-tab="result" data-user="mflerackers" data-pen-title="Animation cubic bézier" class="codepen">See the Pen <a href="https://codepen.io/mflerackers/pen/RYjYgX/">Animation cubic bézier</a> by Marc (<a href="https://codepen.io/mflerackers">@mflerackers</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

$$
a'=t*(b-a)+a \\
c'=t*(c-b)+b \\
d'=t*(d-c)+c \\
a''=t*(c'-a')+a' \\
d''=t*(d'-c')+c' \\
p=t*(d''-a'')+a''
$$

These are a lot of linear interpolations, so we better get the coefficients in order to calculate a simple polynomial. If we write out p in function of a, b, c and d we get the basis functions for a third order Bézier curve

$$
(1-t)^3 \\
3(1-t)^2t \\
3(1-t)t^2 \\
t^3
$$

If we want more bends, or control points, we will have to switch to another strategy, as increasing order will increase the amount of calculations. What we will do is chain several curves. Before we do this, we should actually talk about continuity. If we chain two or more curves, continuity is very important as we don't want gaps or sharp corners. $$C^0$$ continuity means there is no gap where the curves join. This is very easy to achieve, we just set the start point of the next curve to the end point of the previous curve. $$C^1$$ continuity means that we don't change direction abruptly when crossing the point where the curves join. This we can achieve by making sure the handles towards the opposing control points are aligned. We will explore this more in the next chapter where we will talk about derivatives.

Now that we can have a path as long as we want, we'll focus on another problem, namely the fact that control points are handy when you adjust curves by hand, but not when you need to create a path programmatically.

Where do you place them to get the bends you want? To answer this question, we'll look at another family of curves, namely Hermite curves.

## Cubic Hermite spline

An Hermite curve takes two points and two tangent vectors. If we have the Hermite curve defined by $$p_0$$, $$m_0$$, $$m_1$$ and $$p_1$$, then the equivalent Bézier curve has as parameters $$p_0$$, $$p_0+\frac{m_0}{3}$$, $$p_1+\frac{m_1}{3}$$ and $$p_1$$. Thus the tangent vectors can be converted to points and vice versa.

But isn't this a step back? At least we had only points before, we just didn't know where to place them. Now we have tangent vectors, which are less easy to think about. That's true, but we are going to calculate these tangent vectors in function of points on our path. This way we will be able to define our path by the points our path needs to go through, no more parameters which influence our path in a way we don't have full control over.

### Catmull-Rom spline

A Catmull-Rom spline defines the tangent vectors as

$$
m_k=\frac{p_{k+1}-p_{k-1}}{t_{k+1}-t_{k-1}}
$$

In most cases, even spacing is used, so $$t_{k+1}-t_{k-1}$$ can be set to 1 giving

$$
m_k=p_{k+1}-p_{k-1}
$$

### Tension, the Cardinal spline

While a Catmull-Rom has no parameters except for the points we want the path to cross, a cardinal spline introduces one additional parameter, namely tension, denoted as t. This tension parameter, varying between -1 and 1 relaxes the spline or makes it so tense it becomes pointy. Our path still crosses all the points we want, but the shape of the bends is also controllable now.

$$
m_k=(1-t)(p_{k+1}-p_{k-1})
$$

### Bias

Bias, denoted as b, is another useful parameter we can add. It can pull the spline to the previous point or to towards the next point.

$$
m_k=\frac{(1-t)*(1+b)}{2}(p_k-p_{k-1})+\frac{(1-t)*(1-b)}{2}(p_{k+1}-p_k)
$$

### Continuity, the Kochanek-Bartels spline

We can add one more parameter which influences the continuity of the spline, denoted by c. A spline with all 3 parameters is called a Kochanek-Bartels spline.

$$
m_k=\frac{(1-t)*(1-c)*(1+b)}{2}(p_k-p_{k-1})+\frac{(1-t)*(1+c)*(1-b)}{2}(p_{k+1}-p_k)
$$

### The best spline

Which spline to use depends on what your software or game requires. If you make a shape editor, it is probably best to use Bézier splines. If you make an editor for animation paths, Kochanek-Bartels splines give the artist the most flexibility.

If you want procedural animations for your game, a Catmull-Rom spline is probably enough, as you just want a path defined by points, and don't need fine-tuning parameters.

You have to keep in mind that every added parameter increases computation complexity and this means less animated entities to fit into the available CPU time.

## Conversions

While we can choose any of the splines as we see fit, converting between them is sometimes needed. For example for visualization in editing or debugging. Most drawing API's will have quadratic or cubic splines.

As we already saw, an Hermite spline can be written in Bezier form.