---
title: Transformations as matrices
math: true
---

| : ---- : |
| [Previous - Vector projection](3-vector-projection.html) | [Next - Lines](5-lines.html) |

## Why use matrices

Matrices are not a requirement for transforming positions or geometry. Often it is more efficient to build the needed formulas like we did, as allocating, building and multiplying matrices does not come cheap. Where matrices are most handy is when we have an hierarchy of components where a parent component influences the transformation of a child component. In that situation matrices are an indispensable tool.
All the formulas presented here use the OpenGL conventions. This means we will post-multiply matrices and vectors. If for some reason you need matrices for pre-multiplication instead, you only need to transpose the given matrices.

When passing matrices to an API, make sure you know whether it uses row-major or column-major order. Row-major order matrices are stored one row after another. Like the following matrix.

$$\begin{pmatrix}
a_0 & b_1 & c_2 \\
d_3 & e_4 & f_5 \\
g_6 & h_7 & i_8
\end{pmatrix}$$

With the indexes indicating the place where each element is in memory. Column-major order would be stored one column after another.

$$\begin{pmatrix}
a_0 & b_3 & c_6 \\
d_1 & e_4 & f_7 \\
g_2 & h_5 & i_8
\end{pmatrix}$$

Both memory schemes have their advantages. However it has no impact on the actual mathematics. These are only two ways to index a matrix in memory. Code-wise this doesn't have to make a difference if you use named fields. The following code

```cpp
struct Matrix {
	float a, d, g, b, e, h, c, f, i;
};
Matrix m;
float *column_major_matrix = &m.a;
```

would give you a column-major order matrix to pass to OpenGL, while your calculations would keep using the same field names as a row order matrix with sequential fields names.
Just remember that mathematics wise, it doesn't make a difference, OpenGL uses matrices mathematically like they are written here, it just stores it at different indices in memory.

## Matrix transformation

Given the following matrix

$$\begin{pmatrix}
a & b & c \\
d & e & f \\
g & h & i
\end{pmatrix}
\times
\begin{pmatrix}
x \\
y \\
w
\end{pmatrix}$$

The point or vector (x, y, w) can be transformed by calculating

$$x'=x*a+y*b+w*c$$

$$y'=x*d+y*e+w*f$$

$$w'=x*g+y*h+w*i$$

Which is the matrix product of a $$3\times3$$ matrix with a $$1\times3$$ matrix. Matrices can only be multiplied if the amount of columns of the first matrix matches the amount of rows of the second. This is why we write our vector as a column. Keen observers will notice that the elements of our resulting vector are the dot products of the rows of the matrix with the input vector.

$$x'=(a, b, c).(x, y, w)$$

$$y'=(d, e, f).(x, y, w)$$

$$w'=(g, h, i).(x, y, w)$$

We are not going to use general matrices, but are going to focus on linear transformations. In our case, the last row of a matrix {g, h, i} is always {0, 0, 1} so we never need to store this row, and it's values can be hard coded into the multiplication formula.

$$x'=x*a+y*b+w*c$$ 

$$y'=x*d+y*e+w*f$$ 

$$w'=x*0+y*0+w*1=w$$

You might be wondering what the w is doing here Since we are doing 2D, there shouldn't be a third dimension. The w is a convenience in case you want to transform both points and vectors. A point would have w equal to 1, while a vector would use w equal to 0. This makes sure a point is translated and a vector not. Since we are going to transform points, we can keep w equal to 1.

$$x'=x*a+y*b+c$$ 

$$y'=x*d+y*e+f$$

~~~ lua
function Transform:transform(x, y)
    return x * self.a + y * self.b + self.c,
           x * self.d + y * self.e + self.f
end
~~~

## The identity matrix

This matrix does exactly nothing to our input. Multiplying points or vectors with this matrix is like multiplying with 1.

$$\begin{pmatrix}
1 & 0 & 0 \\
0 & 1 & 0 \\
0 & 0 & 1
\end{pmatrix}$$

$$x'=x*1+y*0+0=x$$ 

$$y'=x*0+y*1+0=y$$

~~~ lua
function Transform:__init(values)
    local matrix = values or {1, 0, 0, 0, 1, 0}
    self.a, self.b, self.c, self.d, self.e, self.f = 
        matrix[1], matrix[2], matrix[3], matrix[4], matrix[5], matrix[6]
end
~~~

## The translation matrix

$$\begin{pmatrix}
1 & 0 & tx \\
0 & 1 & ty \\
0 & 0 & 1
\end{pmatrix}$$

$$x'=x*1+y*0+t_x=x+t_x$$

$$y'=x*0+y*1+t_y=y+t_y$$

This is the same as adding the vector $$\langle t_x,t_y\rangle$$ to our point.

~~~ lua
function Transform.Translation(x, y)
    return Transform({1, 0, x, 0, 1, y})
end
~~~

## The rotation matrix

$$\begin{pmatrix}
cos(\alpha) & -sin(\alpha) & 0 \\
sin(\alpha) & cos(\alpha) & 0 \\
0 & 0 & 1
\end{pmatrix}$$

$$x'=x*cos(\alpha)-y*sin(\alpha)+0=x*cos(\alpha)-y*sin(\alpha)$$

$$y'=x*sin(\alpha)+y*cos(\alpha)+0=x*sin(\alpha)+y*cos(\alpha)$$

This is the formula we derived in angles, rotating around the origin.

~~~ lua

function Transform.Rotation(angle)
    local c = math.cos(angle)
    local s = math.sin(angle)
    return Transform({c, -s, 0, s, c, 0, })
end
~~~

## The scale matrix

$$\begin{pmatrix}
sx & 0 & 0 \\
0 & sy & 0 \\
0 & 0 & 1
\end{pmatrix}$$

$$x'=x*s_x+y*0+0=x*s_x$$

$$y'=x*0+y*s_y+0=y*s_y$$

This is multiplication with a scalar.

~~~ lua
function Transform.Scale(x, y)
    y = y or x
    return Transform({x, 0, 0, 0, y, 0})
end
~~~

## Combining transformations

As an example, let's revisit scaling from a given center. We can make a matrix that does this by combining 3 matrices.

$$\begin{pmatrix}
1& 0 & t_x \\
0 & 1 & t_y \\
0 & 0 & 1
\end{pmatrix}
\times
\begin{pmatrix}
s_x& 0 & 0 \\
0 & s_y & 0 \\
0 & 0 & 1
\end{pmatrix}
\times
\begin{pmatrix}
1& 0 & -t_x \\
0 & 1 & -t_y \\
0 & 0 & 1
\end{pmatrix}
\times
\begin{pmatrix}
x \\
y \\
1
\end{pmatrix}$$

Note that since we post-multiply, the matrix that is going to be applied first to the vector, is on the right. The vector follows a transformation path from right to left. In case we were pre-multiplying transformations, the vector would have been be a row matrix on the left, and the order of the matrices would be inverted and the matrices themselves be transposed.

$$\begin{pmatrix}
x & y & 1
\end{pmatrix}
\times
\begin{pmatrix}
1& 0 & 0 \\
0 & 1 & 0 \\
-t_x & t_y & 1
\end{pmatrix}
\times
\begin{pmatrix}
sx& 0 & 0 \\
0 & sy & 0 \\
0 & 0 & 1
\end{pmatrix}
\times
\begin{pmatrix}
1& 0 & 0 \\
0 & 1 & 0 \\
t_x & t_y & 1
\end{pmatrix}$$

Back to our post-multiplied transformation example. Multiplying the first two matrices gives us

$$\begin{pmatrix}
sx& 0 & tx \\
0 & sy & ty \\
0 & 0 & 1
\end{pmatrix}
\times
\begin{pmatrix}
1& 0 & -tx \\
0 & 1 & -ty \\
0 & 0 & 1
\end{pmatrix}$$

Multiplying these we get

$$\begin{pmatrix}
sx& 0 & -sx*tx +tx \\
0 & sy & -sy*ty +ty \\
0 & 0 & 1
\end{pmatrix}$$

Applying this matrix to (x,y) gives us

$$x'=x*s_x+y*0-sx*t_x + t_x$$

$$y'=x*0+y*s_y-s_y*t_y+t_y$$

after grouping the scaled terms, we get the previous formula to scale from a given center

$$x'=s_x*(x-t_x)+t_x$$

$$y'=s_y*(y-t_y)+t_y$$

This formula is of course several magnitudes more efficient than allocating, building and multiplying these three matrices. Only when we don't know in advance what combination of transformations we are going to have, does it make sense to switch to matrices.

## Transforming matrices

Instead of allocating, building and multiplying translation, rotation or scale matrices with our current transformation matrix, we can do better. Since we know the layout of these matrices in advance, we know which elements in our transformation matrix will be modified, and how. This allows us to only do the minimum amount of calculations needed

### Applying a translation to a matrix

Applying a translation only affects the translation of the transform. However the translation itself is modified by the transform's rotation and scale.

$$\begin{pmatrix}
  a & b & c \\
  d & e & f \\
  0 & 0 & 1
 \end{pmatrix}
\times
 \begin{pmatrix}
1& 0 & t_x \\
0 & 1 & t_y \\
0 & 0 & 1
\end{pmatrix}
 =
 \begin{pmatrix}
  a & b & ax+by+c \\
  d & e & dx+ey+f \\
  0 & 0 & 1
 \end{pmatrix}$$

~~~ lua
function Transform:translate(x, y)
    self.c = self.c + x * self.a + y * self.b;
    self.f = self.f + x * self.d + y * self.e;
end
~~~

### Applying a rotation to a matrix

Applying a rotation only affects the rotation and scale part of the transform. It is unaffected by the translation, nor does it modify the translation.

$$\begin{pmatrix}
  a & b & c \\
  d & e & f \\
  0 & 0 & 1
 \end{pmatrix}
\times
 \begin{pmatrix}
cos(\alpha)& -sin(\alpha) & 0 \\
sin(\alpha) & cos(\alpha) & 0 \\
0 & 0 & 1
\end{pmatrix}
 =
 \begin{pmatrix}
  a*cos(\alpha)+b*sin(\alpha) & -a*sin(\alpha)+b*cos(\alpha) & c \\
  d*cos(\alpha)+e*sin(\alpha) & -d*sin(\alpha)+e*cos(\alpha) & f \\
  0 & 0 & 1
 \end{pmatrix}$$

~~~ lua
function Transform:rotate(angle)
    local c = math.cos(angle)
    local s = math.sin(angle)
    self.a, self.b = self.a*c+self.b*s, -self.a*s+self.b*c
    self.d, self.e = self.d*c+self.e*s, -self.d*s+self.e*c
end
~~~

Note that while this code works in lua, other languages might need temporaries.

### Applying a scale to a matrix

Applying a rotation only affects the rotation and scale part of the transform. It is unaffected by the translation, nor does it modify the translation.

$$\begin{pmatrix}
  a & b & c \\
  d & e & f \\
  0 & 0 & 1
 \end{pmatrix}
\times
 \begin{pmatrix}
  x & 0 & 0 \\
  0 & y & 0 \\
  0 & 0 & 1
\end{pmatrix}
 =
 \begin{pmatrix}
  ax & by & c \\
  dx & ey & f \\
  0 & 0 & 1
 \end{pmatrix}$$

~~~ lua
function Transform:scale(x, y)
    y = y or x
    self.a = self.a * x;
    self.d = self.d * x;
    self.b = self.b * y;
    self.e = self.e * y;
end
~~~

## Inverse matrix

The inverse matrix $$M^{-1}$$ is the matrix $$M'$$ for which if $$x'=M \times x$$ then $$x=M' \times x'$$. This matrix is very handy when creating editors for a game, or interaction, as the mouse or finger input needs the inverse camera transform. We'll start with the inverse of the basic transformations. The inverse of the identity matrix is of course the identity matrix again.

### The inverse translation matrix

We only need to translate in the opposite direction, so we switch the sign of the vector.

$${\begin{pmatrix}
1 & 0 & tx \\
0 & 1 & ty \\
0 & 0 & 1
\end{pmatrix}}^{-1}=
\begin{pmatrix}
1 & 0 & -tx \\
0 & 1 & -ty \\
0 & 0 & 1
\end{pmatrix}$$

$$x'=x+(-t_x)$$

$$y'=y+(-t_y)$$

### The inverse rotation matrix

We need to rotate by the angle $$-\alpha$$ instead of $$\alpha$$. Since $$cos(-\alpha)=cos(\alpha)$$ and $$sin(-\alpha)=-sin(\alpha)$$ we get

$${\begin{pmatrix}
cos(\alpha) & -sin(\alpha) & 0 \\
sin(\alpha) & cos(\alpha) & 0 \\
0 & 0 & 1
\end{pmatrix}}^{-1}=
\begin{pmatrix}
cos(\alpha) & sin(\alpha) & 0 \\
-sin(\alpha) & cos(\alpha) & 0 \\
0 & 0 & 1
\end{pmatrix}$$

$$x'=x*cos(\alpha)+y*sin(\alpha)$$

$$y'=-x*sin(\alpha)+y*cos(\alpha)$$

### The inverse scale matrix

We need to inverse the scale factor. If we were scaling two times bigger, we need to scale two times smaller now.

$${\begin{pmatrix}
s_x & 0 & 0 \\
0 & s_y & 0 \\
0 & 0 & 1
\end{pmatrix}}^{-1}=
\begin{pmatrix}
\frac{1}{s_x} & 0 & 0 \\
0 & \frac{1}{s_y} & 0 \\
0 & 0 & 1
\end{pmatrix}$$

$$x'=x*(\frac{1}{s_x})$$

$$y'=y*(\frac{1}{s_y})$$

### The general case

For any general $$3\times3$$ matrix

$$\begin{pmatrix}
  a & b & c \\
  d & e & f \\
  g & h & i
 \end{pmatrix}$$

 for which the determinant is not 0, we can calculate the inverse by calculating

 $$\frac{1}{det}*\begin{pmatrix}
  ei-fg & ch-bi & bf-ce \\
  fg-di & ai-cg & cd-af \\
  dh-eg & bg-ah & ae-bd
 \end{pmatrix}$$

The determinant of a matrix can be calculated by adding the product of the left-top to right-bottom diagonals and subtracting the products of the left-bottom to top-right diagonals. Another way is taking the sum of the top row elements multiplied by the determinants of the $$2\times2$$ matrices obtained by removing the row and columns from which said elements are part. Either way gives us as determinant 

$$det=a(ei - fh) - b(di - fg) + c(dh - eg)$$ 

That's quite a lot of calculations for obtaining the inverse of a matrix. Luckily our transformation matrices are more specific, as the last row is always 

$$\begin{vmatrix}0 & 0 &1\end{vmatrix}$$

If we take this into consideration, by replacing g, h, i with 0, 0, 1 respectively we get the following much simplified solution

 $$\begin{pmatrix}
  a & b & c \\
  d & e & f \\
  0 & 0 & 1
 \end{pmatrix}^{-1}=\frac{1}{det}*\begin{pmatrix}
 e & -b &  bf - ce \\
 -d & a & cd - af \\
 0 & 0 & ae-bd
 \end{pmatrix}$$

The determinant can be simplified as well

$$a*e - b*d$$

Note that element i will always be 1 after division with the determinant. Also note that if there is no scale in the matrix, but only translation and rotation, the determinant will be 1.

~~~ lua
function Transform:inverse()
    local det = self.a*self.e-self.b*self.d
    if det < 0.00001 and det > -0.00001 then return nil end
    return Transform{
      self.e/det, -self.b/det, (self.b*self.f-self.c*self.e)/det,
      -self.d/det, self.a/det, (self.c*self.d-self.a*self.f)/det
    }
end
~~~

## Extracting the translation, rotation and scale factors

### Extracting the translation

This is very easy, as the translation has its own place in the matrix.

$$\begin{pmatrix}
  a & b & \textbf{c} \\
  d & e & \textbf{f} \\
  0 & 0 & 1
 \end{pmatrix}$$

~~~ lua
function Transform:getTranslation()
    return self.c, self.f
end
~~~

### Extracting the rotation

The rotation is also still fairly easy, since we just need to get the sine and cosine. These are available in a, d, and b, e, we can use them with the arc tangents to obtain the angle. 

$$\begin{pmatrix}
  \textbf{a} & \textbf{b} & c \\
  \textbf{d} & \textbf{e} & f \\
  0 & 0 & 1
 \end{pmatrix}$$

Note that we strategically take a and d, not for example a and b. This is because both a and d are multiplied by the same factor. Since the arc tangent uses the slope, or d divided by a, this factor is eliminated.

$$\begin{pmatrix}
  x*cos & -y*sin & c \\
  x*sin & y*cos & f \\
  0 & 0 & 1
 \end{pmatrix}$$

~~~ lua
function Transform:getRotation()
    return math.atan2(self.d, self.a)
end
~~~

### Extracting the scale

The scale poses a bit of a problem as we actually need to erase the rotation. Element a is equal to $$x*cos(\alpha)$$, while d is equal to $$x*sin(\alpha)$$. What we can do is make use of the fact that $$\\cos(\alpha)^2+sin(\alpha)^2=1$$ since $$\langle cos, sin\rangle$$ is a normalized vector. If we write out $$a^2+d^2$$, we have

$$a^2+d^2=(x*cos(\alpha))^2+(x*sin(\alpha))^2$$

Which we can write as

$$a^2+d^2=x^2*cos(\alpha)^2+x^2*sin(\alpha)^2$$

Extracting the common factor we get

$$a^2+d^2=x^2*(cos(\alpha)^2+sin(\alpha)^2)=x^2*1$$

So if we take the square root of the above formula we get

$$\sqrt{a^2+d^2}=x$$

~~~ lua
function Transform:getScale()
    return math.sqrt(self.a * self.a + self.d * self.d)
           math.sqrt(self.b * self.b + self.e * self.e)
end
~~~

## Applications

### Transforming objects

If we want to transform an object by giving it a translation, rotation and scale, we need to do these individual transformations in the correct order. Two translations can be done in any order, as they are commutative. The same can be said for two rotations or two scales. 

If however we do a translation after a scale, the translation will be influenced by the scale. Similarly if we scale before we rotate, the scale will use the old x and y axis instead of the rotated ones.

The correct order, for transforming an object, is to do the translation first, then the rotation and finally the scale.

```lua
function Object:transform()
  translate(self.x, self.y)
  rotate(self.angle)
  scale(self.scale, self.scale)
end
```

Vertices emitted after these transformations will be first scaled in their local coordinate system. Then they will be rotated, and finally translated to their global position.

If we want a flexible anchor for scaling or rotating around, we need to add a translation so that the anchor becomes the local origin before scaling. We need to undo this translation afterwards so that our object is positioned correctly.

```lua
function Object:transform()
  translate(self.x+self.anchorX, self.y+self.anchorY)
  rotate(self.angle)
  scale(self.scale, self.scale)
  translate(-self.anchorX, -self.anchorY)
end
```

As you see we can combine our undo of the anchor with the positioning of the object, saving one transformation.

### Hierarchical transformations

In a lot of cases, whether using transformations in a game scene, an animated character, or a GUI, there is a certain hierarchy of components where the transformation of a parent component influences the transformation of its child components. For example moving a window, will move its buttons as well.

When we look at one component, for example a button, we talk about its local transformation inside its parent, for example 10 pixels right and 20 down from the left top of the window, and its global transformation, the distance from the left top of the screen.

Now the question is, at drawing time, how do we get the global transformation to transform our vertices?

We could of course travel all the way to the parent, then travel the path in reverse while multiplying the local transformations of every component we meet, but that would mean all lot of overhead by multiplying the same transformations for every component.

We could travel the tree depth first starting from the root, and store the global matrix for each component. While that might cost us quite some additional memory, having the transformation matrix stored can be handy for inverse transformations in the case of clicking or selection.

If we want a more flexible system, we can use a transformation stack. We can pre-allocate this stack to the maximum depth we expect, and place an identity matrix at the top. When we want to do some transformations in a component, we first push a copy of the current stack top onto the stack, do our modifications and pop the copy from the stack when we're done. Note that no actual allocations or deallocations happen as we merely change the pointer or index indicating the current top.

```lua
local TransformStack = class()
function TransformStack:__init(n)
    n = n or 32
    self._stack = {}
    for i=1,n do table.insert(self._stack, Transform()) end
    self._index = 1
    self._top = self._stack[self._index]
end

function TransformStack:push()
    if self._index == #self._stack then error("Transform stack is full") end
    local from = self._stack[self._index].matrix
    local to = self._stack[self._index+1].matrix
    for i=1,6 do to[i] = from[i]; end
    self._index = self._index + 1
    self._top = self._stack[self._index]
end

function TransformStack:pop()
    if self._index == 1 then error("Transform stack is empty") end
    self._index = self._index - 1
    self._top = self._stack[self._index]
end

function TransformStack:top()
    return self._top
end
```

### Camera transformations in games

Since we want the center of our camera view on the center of the screen, we do an initial translation.

```lua
function Camera:transform()
  translate(screenCenterX,screenCenterY)
end

function Camera:transformPoint(x,y)
  return x+screenCenterX, y+screenCenterY
end
```

#### Panning

We pan by translating the world in the inverse direction of the camera. If the camera goes left, the world moves right relative to the camera.

```lua
function Camera:moveTo(x,y)
  self.x = x
  self.y = y
end

function Camera:moveBy(x,y)
  self.x = self.x + x
  self.y = self.y + y
end

function Camera:transform()
  translate(screenCenterX-self.x, screenCenterY-self.y)
end

function Camera:transformPoint(x,y)
  return x+screenCenterX-self.x, y+screenCenterY-self.y
end
```

#### Zooming

If we want to zoom in, we use a scale factor greater than 1, making everything bigger. For zooming out, we use a scale smaller than 1, making everything smaller on the screen.

If we translate the camera as well, this needs to be done before scaling, which in case of post multiplied matrices means multiplying the translation matrix last.

```lua
function Camera:zoomTo(s)
  self.zoom = s
end

function Camera:zoomBy(s)
  self.zoom = self.zoom * s

function Camera:transform()
  translate(screenCenterX, screenCenterY)
  scale(self.zoom)
  translate(-self.x, -self.y)
end

function Camera:transformPoint(x,y)
  return screenCenterX+self.zoom*(x-self.x), screenCenterY+self.zoom*(y-self.y)
end
```

#### Rotating

If we need rotation as well, we do this after translating and before scaling. In our case, since we probably scale uniformly, thus x and y by the same factor, the order of rotation and scale isn't that important, however if you use different factors for x and y, you need to be careful.

```lua
function Camera:rotateTo(angle)
  self.angle = angle
end

function Camera:rotateBy(angle)
  self.angle = self.angle + angle

function Camera:transform()
  translate(screenCenterX, screenCenterY)
  scale(self.zoom)
  rotate(-self.angle)
  translate(-self.x, -self.y)
end

function Camera:transformPoint(x,y)
  x, y = x-self.x, y-self.y
  local c = math.cos(-self.angle)
  local s = math.sin(-self.angle)
  return screenCenterX + self.zoom*(c*x-s*y), screenCenterY + self.zoom*(s*x+c*y)
end
```

### Selection

Let's pretend we are making an RTS game, and we want to select units using the mouse. This is easy when world and screen space match, but what if the camera allows us to pan around the screen, zoom out or even rotate the world to see what's behind those trees?

A naive thought would be to transform all units one by one to screen space and test  whether our mouse coordinates, which are given in screen space, are inside the transformed collision shapes, be it circles, triangles, boxes or polygons.

A much better way would be to just transform our mouse coordinates to world space, since it only requires one point transform. This is where the inverse transform comes into play, as we only have the camera transformation from world to screen, in order to transform from screen to world, we need to calculate the inverse transformation. Note that it is still faster than transforming many objects to screen space. Also, if the camera isn't moving during selection, the inverse doesn't need to be calculated every frame.

```lua
function love.mousepressed(x, y, button, istouch)
  x, y = stack.top().inverse().transform(x, y)
  -- Check for all selectable entities whether they intersect with x, y
end
```

| : ---- : |
| [Previous - Vector projection](3-vector-projection.html) | [Next - Lines](5-lines.html) |
