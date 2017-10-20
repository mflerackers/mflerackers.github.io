---
layout: default
title: Transformations as matrices
math: true
---

| : ---- : |
| [Previous - Vector projection](3-vector-projection.html) | [Next - Lines](5-lines.html) |

# [Math for game development](../) : Transformations as matrices

# Transformations as matrices

Matrices are not a requirement for transforming positions or geometry. Often it is more efficient to build the needed formulas like we did, as building and multiplying matrices does not come cheap. Where matrices are most handy is when we have a hierarchy of components where a parent component influences the transformation of a child component. In that situation matrices are an indispensable tool.
All the formulas presented here use the OpenGL conventions. This means we will post-multiplied matrices and vectors. If for some reason you need matrices for pre-multiplication instead, you only need to transpose the given matrices.
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

We are not going to use general matrices, but are going to focus on linear transformations. In our case, the last row of a matrix {g, h, i} is always {0, 0, 1} so we never need to store this row, and it's values can be hard coded into multiplication formula.

$$x'=x*a+y*b+w*c$$ 

$$y'=x*d+y*e+w*f$$ 

$$w'=x*0+y*0+w*1=w$$

You might be wondering what the w is doing here Since we are doing 2D, there shouldn't be a third dimension. The w is a convenience in case you want to transform both points and vectors. A point would have w equal to 1, while a vector would use w equal to 0. This makes sure a point is translated and a vector not. Since we are going to transform points, we can keep w equal to 1.

$$x'=x*a+y*b+c$$ 

$$y'=x*d+y*e+f$$

## The identity matrix

This matrix does exactly nothing to our input. Multiplying points or vectors with this matrix is like multiplying with 1.

$$\begin{pmatrix}
1 & 0 & 0 \\
0 & 1 & 0 \\
0 & 0 & 1
\end{pmatrix}$$

$$x'=x*1+y*0+0=x$$ 

$$y'=x*0+y*1+0=y$$

## The translation matrix

$$\begin{pmatrix}
1 & 0 & tx \\
0 & 1 & ty \\
0 & 0 & 1
\end{pmatrix}$$

$$x'=x*1+y*0+t_x=x+t_x$$

$$y'=x*0+y*1+t_y=y+t_y$$

## The rotation matrix

$$\begin{pmatrix}
cos(\alpha) & -sin(\alpha) & 0 \\
sin(\alpha) & cos(\alpha) & 0 \\
0 & 0 & 1
\end{pmatrix}$$

$$x'=x*cos(\alpha)-y*sin(\alpha)+0=x*cos(\alpha)-y*sin(\alpha)$$

$$y'=x*sin()\alpha+y*cos(\alpha)+0=x*sin()\alpha+y*cos(\alpha)$$

## The scale matrix

$$\begin{pmatrix}
sx & 0 & 0 \\
0 & sy & 0 \\
0 & 0 & 1
\end{pmatrix}$$

$$x'=x*s_x+y*0+0=x*s_x$$

$$y'=x*0+y*s_y+0=y*s_y$$

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

after grouping the scaled terms, the previous formula to scale from a given center

$$x'=s_x*(x-t_x)+t_x$$

$$y'=s_y*(y-t_y)+t_y$$

This formula is of course several magnitudes more efficient than building and multiplying these three matrices. Only when we don't know in advance what combination of transformations we are going to have to do, does it make sense to switch to matrices.

## Inverse matrix

The inverse matrix $M^{-1}$ is the matrix $M'$ for which if $x'=M \times x$ than $x=M' \times x'$. This matrix is very handy when creating editors for a game, or interaction, as the mouse or finger input needs the inverse camera transform. We'll start with the inverse of the basic transformations. The inverse of the identity matrix is of course the identity matrix again.

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

We need to rotate by the angle $-\alpha$ instead of $\alpha$. Since $cos(-\alpha)=cos(\alpha)$ and $sin(-\alpha)=-sin(\alpha)$ we get

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

$$y'=-x*sin()\alpha+y*cos(\alpha)$$

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

For any general $3\times3$ matrix

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

The determinant of a matrix can be calculated by adding the product of the left-top to right-bottom diagonals and subtracting the products of the left-bottom to top-right diagonals. Another way is taking the sum of the top row elements multiplied by the determinants of the $2\times2$ matrices obtained by removing the row and columns from which said elements are part. Either way gives us as determinant 

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

## Extracting the translation, rotation and scale factors