---
layout: single
title:  "Fake voxel rendering"
date:   2018-08-14 11:10:01 +0900
categories: math
excerpt: "A short explanation on how to render fake voxels for 2D games."
---

## Rendering an individual object

A fake voxel object is rendered as a stack of sprites. The bottom sprite is drawn at the position of the object, every next sprite is drawn a few pixels above the previous one. Most logically, the distance between sprites is the same as the size of a voxel. So if you scale up sprites by 4, you offset each next sprite by 4 pixels.
Each sprite is drawn rotated, the angle is the sum of the camera angle and the angle of the object, the origin is the center of the sprite.

```lua
local offset = self.y
local angle = self.angle + camera.angle
for _, layer in ipairs(self.layers) do
    love.graphics.draw(self.image, layer, self.x, offset, angle, 4, 4, self.ox, self.oy)
    offset = offset-4
end
```

## Transforming objects : translating the camera

If we want to roam the world by translating the camera, we need to transform each object  (technically we could put the translation on the transformation stack and let the vertex shader do it, but since we'll be doing more than just translating later on, we do it on the CPU).
Our camera has a position $$(cx, cy)$$, and we want to make sure the center of our world ends up in the middle of the screen $$(sx, sy)$$. So our transformation is

$$
\begin{pmatrix}
  1 & 0 & sx - cx \\
  0 & 1 & sy - cy \\
  0 & 0 & 1
 \end{pmatrix}
$$
 
This translates the point in the world at $$(cx, cy)$$ to the middle of the screen $$(sx, sy)$$. Applying this matrix to the position of the object is nothing but a vector addition, since we are not scaling or rotating yet.

## Transforming objects : rotating the camera

First note that when we talk about the camera angle $$\alpha$$ here, it is actually the reverse angle, because just like the translation, when the camera rotates clockwise, the world which we see actually rotates counterclockwise.

When we rotate the camera, we want it to rotate not around $$(0, 0)$$, but the place where the camera is. So we actually need to translate, rotate and translate back (like you do when rotating a sprite around a different origin than its left top). So our rotation around $$(cx, cy)$$ is

$$
\begin{pmatrix}
  cos(\alpha) & -sin(\alpha) & cx - cos(\alpha) * cx + sin(\alpha) * cy \\
  sin(\alpha) & cos(\alpha) & cy - sin(\alpha) * cx - cos(\alpha) * cy \\
  0 & 0 & 1
 \end{pmatrix}
 $$

 When we combine this with our camera translation from before, we multiply these matrices (which ends up to be just an addition of the translation vectors since our translation is independent of the rotation) and get

 $$
 \begin{pmatrix}
  cos(\alpha) & -sin(\alpha) & sx - cx + cx - cos(\alpha) * cx + sin(\alpha) * cy \\
  sin(\alpha) & cos(\alpha) & sy - cy + cy - sin(\alpha) * cx - cos(\alpha) * cy \\
  0 & 0 & 1
 \end{pmatrix}
 $$

 Or after simplifying

$$
  \begin{pmatrix}
  cos(\alpha) & -sin(\alpha) & sx - cos(\alpha) * cx + sin(\alpha) * cy \\
  sin(\alpha) & cos(\alpha) & sy - sin(\alpha) * cx - cos(\alpha) * cy \\
  0 & 0 & 1
 \end{pmatrix}
$$

The two left rows will rotate our position, and the right row will offset it by the screen and the camera position. Since our camera is expressed in coordinates of the world, which is rotated now, the camera vector has been rotated as well. If you don't see it yet, we can rewrite

$$
sx - cos(\alpha) * cx + sin(\alpha) * cy\\sy - sin(\alpha) * cx - cos(\alpha) * cy
$$

as

$$
sx - (cos(\alpha) * cx - sin(\alpha) * cy)\\sy - (sin(\alpha) * cx + cos(\alpha) * cy)
$$

which shows it as being an offset composed of the screen offset subtracted by the camera offset which is first rotated by the camera angle.

In this final transformation, we can notice the following

1. It contains cosine and sine, we don't want to calculate these often, so we better cache them.
2. The last row is a vector which is only dependent on the camera rotation and position, as long as the camera doesn't change don't change, we can cache this too.

Given this, we can write our transform function, which transforms a world position to a screen position (in other words, which tells us where to draw an object) as follows

```lua
function Camera:transform(x, y)
    return x * self.cos - y * self.sin + self.offsetX,
           x * self.sin + y * self.cos + self.offsetY
end
```

Where cos, sin and the offset vector are only calculated when the viewpoint changes

```lua
function Camera:recalc(angleChanged)
    if angleChanged then
        self.cos = math.cos(self.angle)
        self.sin = math.sin(self.angle)
    end
    self.offsetX = self.screenX - self.cos * self.x + self.sin * self.y
    self.offsetY = self.screenY - self.sin * self.x - self.cos * self.y
end
```

## Moving around

Rotating the camera is easy, we just need to change the angle. To pan the camera however, is a bit more involved as we want it to always pan left-right and up-down according to the screen, not how the camera is rotated, as it will confuse the player.

This means rather than adding $$(0, -speed * dt)$$ to the camera position when pressing up (remember, when the camera goes up, the world goes down), we need to counter rotate the vector $$(speed * dt, 0)$$ first, so it is not influenced by the camera angle.

So we need the following transformation matrix

$$
\begin{pmatrix}
cos(-\alpha) & -sin(-\alpha) & 0 \\
sin(-\alpha) & cos(-\alpha) & 0 \\
0 & 0 & 1
\end{pmatrix}
$$

Unfortunately, we need the cosine and sine of $$-\alpha$$, while we only have the cosine and sine of $$\alpha$$. Lucky for us there's the following identity

$$
cos(-\alpha)=cos(\alpha)\\
sin(-\alpha)=-sin(\alpha)
$$

so we can use

$$
\begin{pmatrix}
cos(\alpha) & sin(\alpha) & 0 \\
-sin(\alpha) & cos(\alpha) & 0 \\
0 & 0 & 1
\end{pmatrix}
$$

for which we have everything cached already.
Transforming our camera displacement $$(dx, dy)$$ by this matrix gives us

$$
cx' = cx + dx*cos(\alpha) + dy*sin(\alpha)\\
cy' = cy - dx*sin(\alpha) + dy*cos(\alpha)
$$

For the up key, which uses $$(0, -speed * dt)$$ as $$(dx, dy)$$ that gives

$$
cx' = cx - speed * dt *dy*sin(\alpha)\\
cy' = cy - speed * dt *dy*cos(\alpha)
$$

For the other direction, down $$(0, speed * dt)$$, or strafing left $$(-speed * dt, 0)$$ and right $$(speed * dt, 0)$$ we can do the same.

## Intermezzo : dropping the cosine and sine calculations

While they are by no means taxing calculations since they are only done while rotating the camera, we can in fact remove them. With one caveat however, we won't be able to rotate arbitrary angles.

Why is it important to be able to rotate by an arbitrary angle? Mainly because in most cases, our time between updates will not be exactly fixed, like 1/60th of a second. The frame rate might be a lot higher on a PC, and lower on mobile. This means that if you would turn each update at a fixed angle, the rotation speed will be linked to the speed of the device. Since we don't want that, we usually multiply the angular speed by the time it took since the last update.

But let's say we are OK with fixed angles. We might for example accumulate time until we have enough to rotate a bit, or might do multiple fixed rotations to cover the time on a slow device.

We have the current angle $$\alpha$$ and want to add the fixed angle $$\beta$$. This means we need to calculate $$cos(\alpha+\beta)$$ and $$sin(\alpha+\beta)$$, but we don't want to actually call the cosine and sine functions. What we do have is $$cos(\alpha)$$ and $$sin(\alpha)$$ since we were using it for the last update, and another handy identity

$$
cos(\alpha+\beta)=cos(\alpha) * cos(\beta) − sin(\alpha) * sin(\beta) \\
sin(\alpha+\beta)=sin(\alpha) * cos(\beta) + cos(\alpha) * sin(\beta)
$$

So if we just calculate $$cos(\beta)$$ and $$sin(\beta)$$ at load time, we can use it to rotate the current $$cos(\alpha)$$ and $$sin(\alpha)$$ in any direction (remember the $$-\alpha$$ identity from above) by $$\beta$$ without using cosine or sine during run-time.

## Z-ordering

Now that we can move and rotate our camera around, we notice that objects are not always drawn in the right order. Sometimes a further object obscures a closer one. This is because we  draw our objects in the order they were placed in the list. We actually need to draw them from back to front. So we will need to sort our list before drawing.

For this we need to split the transform and draw stages. First we transform all objects and update their screen coordinates. Then we sort the list and finally we draw them in the order of the newly sorted list.

This example uses lua's built-in table.sort which uses quicksort internally. While quicksort is not a bad algorithm, it has the disadvantage that it has a worst case of $$O(n)$$ and that it is not a stable sort. Not being stable means that two objects at the exact same y may switch places after a second sort, which may cause flicker.

Another thing to note is that, while we rotate, not all objects should be re-sorted. Depending on the distance between objects, their order stays the same for many angles.
A better sort algorithm would be for example [timsort](http://svn.python.org/projects/python/trunk/Objects/listsort.txt) which is a stable sort algorithm based on mergesort with a worst case of $$O(n\log{}n)$$. It can cope much better with partially sorted lists.

LuaJit has been planning to move to [smoothsort](https://en.wikipedia.org/wiki/Smoothsort), a sort algorithm with a worst case of $$O(n\log{}n)$$ that is based on heapsort. This makes it good for partially sorted data, but like heapsort it is not a stable sort.

## Picking : inverse transform

If we want to build an editor (of course we want to) for our stages, we need to be able to place objects. We can do this easily when the camera is not rotated, we just need to subtract the center of the screen, since this is where the camera is drawn and add the camera position, since the camera is placed at $$(0, 0)$$ after subtracting the screen center.
What we did just now is building a transformation from screen to world coordinates. 

$$
\begin{pmatrix}
  1 & 0 & - sx + cx \\
  0 & 1 & - sy + cy \\
  0 & 0 & 1
\end{pmatrix}
$$

This was easy, but once we add a rotation, things get more complex. Let's look at the general solution for an inverse matrix first.

### General inverse

For any general $$3\times3$$ matrix

$$
\begin{pmatrix}
  a & b & c \\
  d & e & f \\
  g & h & i
\end{pmatrix}
$$

for which the determinant is not 0, we can calculate the inverse by calculating

$$
\frac{1}{det}*\begin{pmatrix}
  (ei-fg) & -(bi-ch) & (bf-ce) \\
  -(di-fg) & (ai-cg) & -(af-cd) \\
  (dh-eg) & -(ah-bg) & (ae-bd)
 \end{pmatrix}
$$

The determinant of a matrix can be calculated by adding the product of the left-top to right-bottom diagonals and subtracting the products of the left-bottom to top-right diagonals. Another way is taking the sum of the top row elements multiplied by the determinants of the $$2\times2$$ matrices obtained by removing the row and columns from which said elements are part. Either way gives us as determinant

$$
det=a(ei−fh)−b(di−fg)+c(dh−eg)
$$

That's quite a lot of calculations for getting an inverse. Luckily our transformation matrices are more specific, as the last row is always

$$
\begin{vmatrix}
0 & 0 & 1
\end{vmatrix}
$$

If we take this into consideration, by replacing g, h, i with 0, 0, 1 respectively we get the following much simplified solution

$$
\begin{pmatrix}
  a & b & c \\
  d & e & f \\
  0 & 0 & 1
\end{pmatrix}^{-1}=\frac{1}{det}*\begin{pmatrix}
 e & -b & bf−ce \\
 -d & a & cd−af \\
  0 & 0 & ae−bd
 \end{pmatrix}
$$

The determinant can be simplified as well

$$
a∗e−b∗d
$$

In the case of our world to screen and screen to world transformation the determinant will 1, as we don't scale or shear. The determinant is an indicator of how the transformation influences the area between 3 or more points. Just translating and rotating doesn't change the area, so the determinant is 1.

We can now fill in the values of our world to screen matrix to calculate the inverse.

However instead of doing this right now, we're going to approach the problem with some logic instead and develop the inverse transformation ourselves. While solutions like we just saw are handy for general cases , they don't help us to develop a deeper understanding of what is going on.

### Getting to the inverse step by step 

Our world to screen transform is as follows

$$
\begin{pmatrix}
  cos(\alpha) & -sin(\alpha) & sx - cos(\alpha) * cx + sin(\alpha) * cy \\
  sin(\alpha) & cos(\alpha) & sy - sin(\alpha) * cx - cos(\alpha) * cy \\
  0 & 0 & 1
 \end{pmatrix}
$$

It takes a point in world coordinates and subtracts the camera position from it, thus bringing the camera to $$(0, 0)$$. Then it rotates everything around $$(0, 0)$$ and finally the screen center or camera screen position $$(sx, sy)$$ is added to bring the camera to the middle of the screen. This sequence can be written as the multiplication of three matrices

$$
\begin{pmatrix}
  1 & 0 & sx \\
  0 & 1 & sy \\
  0 & 0 & 1
 \end{pmatrix} * 
 \begin{pmatrix}
  cos(\alpha) & -sin(\alpha) & 0 \\
  sin(\alpha) & cos(\alpha) & 0 \\
  0 & 0 & 1
 \end{pmatrix} * 
 \begin{pmatrix}
  1 & 0 & -cx \\
  0 & 1 & -cy \\
  0 & 0 & 1
 \end{pmatrix}
$$

To go from screen to world, we need to do everything in reverse. Both the order, and every individual transformation.

$$
\begin{pmatrix}
  1 & 0 & cx \\
  0 & 1 & cy \\
  0 & 0 & 1
 \end{pmatrix} * 
 \begin{pmatrix}
  cos(\alpha) & sin(\alpha) & 0 \\
  -sin(\alpha) & cos(\alpha) & 0 \\
  0 & 0 & 1
 \end{pmatrix} * 
 \begin{pmatrix}
  1 & 0 & -sx \\
  0 & 1 & -sy \\
  0 & 0 & 1
 \end{pmatrix}
$$

The inverse of a translation is translating in the opposite direction, so we just flip signs.
The inverse of a rotation is rotating in the opposite direction. This means we just need to flip the signs of the sines.
If we combine these matrices again, we get the following inverse transformation matrix.

$$
\begin{pmatrix}
  cos(\alpha) & -sin(\alpha) & cx - cos(\alpha) * sx - sin(\alpha) * sy \\
  sin(\alpha) & cos(\alpha) & cy + sin(\alpha) * sx - cos(\alpha) * sy \\
  0 & 0 & 1
 \end{pmatrix}
$$

This is the same result we get when filling in our world to screen matrix elements into the formula to calculate the inverse. The screen position of the camera gets a rotation, and the rotation of the world position of the camera gets cancelled out because

$$
cos^2(\alpha) + sin^2(\alpha) = 1
$$

## Clipping : only drawing what we see

Until now we were drawing all objects, whether they were within the screen area or not. If our world or stage is a lot bigger than the screen, we should skip drawing objects which are not visible in order to reduce or CPU usage.

There are several ways to approach this. The smallest change to our code would be to check whether an object is within the screen rectangle, after calculating screen coordinates and before sorting.

But this means we would still be wasting calculations on transforming all objects to the screen.

Since we have the inverse transform as well, we can instead transform the four screen corners of the screen to the world. From those we can construct a new axis aligned (in the world's coordinate system) rectangle with which we can check whether objects are within the screen by using just their world coordinates.

This is better, but if we have a large world, we're still doing a lot of testing. We can avoid this by subdividing our world in smaller sections. Instead of one big list, we give each section a list of objects which are situated in that section. This allows us to only test whether a section is within the screen, and discard many objects at once if it isn't.
Care should be taken that we do not make our world less organic because of the limits we impose on it. If we use a grid to cut our world in sections for example, an object might be crossing a border. This means that if that object is not allowed to be in both sections, it will disappear when the section it is in is not visible. Placing it in both grid squares is an option, but then care should be taken not to place it twice in the drawing list, which might incur additional processing (using a set instead of a list).

## Drawing sprites in batches

Finally, drawing each sprite separately when using a GPU isn't very good for performance. Instead we should place all sprites into a batch and draw them all at once.

You can find three renderers in the example code. One which draws directly, and two which batch drawing calls. One batch renderer only supports a single texture, the other one supports as many textures as needed. Note that both are primitive though. Usually you start a new batch when your draw call needs to be flushed. This is when a textures, blending mode, shader or other non uniform variable is changed.

For the code, see the [github](https://github.com/mflerackers/fakevoxels) repository.