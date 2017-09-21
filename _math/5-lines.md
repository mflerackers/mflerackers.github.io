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

$$p=a+s\vec{ab}$$

For every scalar s the point p will be a poimt on our line.
