---
layout: single
title:  "Introduction to statistics: Part 1"
date:   2018-05-11 22:03:01 +0900
categories: math
excerpt: "A short introduction to statistical entities and how to use them. From mean to principal component analysis."
toc: true
---

## Our data set

For doing statistical calculations we need data. We will create some fake data points here to use as an example. We're growing plants and giving them a certain amount of sunlight. We measure the growth after a certain interval and place all this data into a table.

| Light (hours)| Growth (mm) |
| -------------| ----------- |
| 0            | 0           |
| 1            | 1           |
| 2            | 5           |
| 3            | 9           |
| 4            | 12          |
| 5            | 15          |

## Mean

The mean or average is calculated by dividing the sum of all samples by the amount of samples.

$$
\overline{x}=\frac{\sum\limits_{i=1}^{n} x_i}{n}
$$

This is the population mean

In case we only have samples, rather than a complete population, we need to divide by $$n-1$$ instead of $$n$$.

$$
\overline{x}=\frac{\sum\limits_{i=1}^{n} x_i}{n-1}
$$

This is called the sample mean.

For our plant growth, the mean of the plant growth is (0+1+5+9+12+15)/6 or 42/6 which is 7.

## Median

The median is calculated by sorting the samples and taking the middle sample. In case the amount of samples n is even, the median of the two middle samples is taken.

In our case, the median of the plant growth is (5+9)/2 or 7, which is the mean growth of 2 and 3 hours of light.

## Variance

The variance is a measure of how much variance or difference there is among our samples. A low variance means most samples are equal, a high variance means there are many samples lower or higher than the median.

The variance is calculated by summing the square of the difference between each sample and the mean, and dividing it by $$n-1$$.

$$
\frac{\sum\limits_{i=1}^{n} (x_i-\overline{x})^2}{n-1}
$$

Thus it is the sample mean of the squared differences of each sample $$x$$ with the mean $$\overline{x}$$.

Conceptually, the variance is the average squared distance of a sample to the mean.

To calculate the variance of the plant growth we take all differences -7, -6, -2, 2, 5, 8 and square them to obtain 49, 36, 4, 4, 25, 64. We add all of these together and divide them by 5 to get (49+36+4+4+25+64)/5 or 36.4.

## Standard deviation

If we take the square root of the variance, we get the standard deviation. Since we take the square root of the squared average distance to the mean, this is the average distance of a sample to the mean.

$$
\sqrt{\frac{\sum\limits_{i=1}^{n} (x_i-\overline{x})^2}{n-1}}
$$

The square root of the variance of our plant growth is $$\sqrt{36.4}$$ or 6.03, about 6. Since our values of growth go from 0 to 15, a standard deviation of 6 means there is quite some difference between our plant growth samples. It means that the average distance from the mean growth of 7mm is 6mm.

## Covariance

Until now, we worked with one measurement, $$x$$. If we want to compare an input parameter with our measurement, we use the covariance. The covariance is calculated by taking the sum of the products of the differences of each sample with its mean, divided by n-1. We basically swapped one $$(x_i-\overline{x})$$ with $$(y_i-\overline{y})$$.

$$
\frac{\sum\limits_{i=1}^{n} (x_i-\overline{x})(y_i-\overline{y})}{n-1}
$$

The covariance when calculated for the amount of light and plant growth is 11.2. To calculate this, we calculated the mean of our hours of light, which is 2.5. Then we took the sum of all products of the differences between light and light mean and growth and growth mean, finally dividing by 5.

## Linear regression and the regression coefficient

When we have two sets of samples, we can calculate a line which follows the trend of the samples. This line can be used to extrapolate or guess values for which we don't have samples or just to view the trend.

A line can be defined as an equation $$y=ax+b$$, where a is the slope and b is the position where the line crosses the x axis, thus where y is 0.

One way to calculate the slope a is by the dividing the covariance of x and y by the variance of x.

$$
\frac{cov(x, y)}{var(x)}
$$

This is called the regression coefficient.

If we don't have the covariance and variance yet we can use the following formula instead

$$
a=\frac{n\sum\limits_{i=1}^{n}{x_iy_i}-\sum\limits_{i=1}^{n}{y_i}\sum\limits_{i=1}^{n}{x_i}}{n\sum\limits_{i=1}^{n}{x_i^2}-\sum\limits_{i=1}^{n}{_ix}\sum\limits_{i=1}^{n}{x_i}}
$$

Both formulas are mathematically identical, though limits with floating point values can give slightly different results.

Once we have a, we can calculate b since

$$
\frac{\sum\limits_{i=1}^{n}{y_i}}{n}=a\frac{\sum\limits_{i=1}^{n}{x_i}}{n}+b
$$

We can subtract $$a\frac{\sum\limits_{i=1}^{n}{x_i}}{n}$$ from both sides to get

$$
b=\frac{\sum\limits_{i=1}^{n}{y_i}}{n}-a\frac{\sum\limits_{i=1}^{n}{x_i}}{n}
$$

or if we group the fractions

$$
b=\frac{\sum\limits_{i=1}^{n}{y_i}-a\sum\limits_{i=1}^{n}{x_i}}{n}
$$

## The Pearson correlation coefficient R

The correlation coefficient is to covariance as the standard deviation is to variance. It is a better measure to interpret results from.
The correlation coefficient is calculated by dividing the covariance by the product of the standard deviations of each measurement.

$$
\frac{cov(x,y)}{std(x)std(y)}=\frac{\frac{\sum\limits_{i=1}^{n} (x_i-\overline{x})(y_i-\overline{y})}{n-1}}{\sqrt{\frac{\sum\limits_{i=1}^{n} (x_i-\overline{x})^2}{n-1}}\sqrt{\frac{\sum\limits_{i=1}^{n}(y_i-\overline{y})^2}{n-1}}}
$$

However if we place the square root outside, we see that correlation coefficient is equal to the square root of the product of the regression coefficients for both samples

$$
\sqrt{\frac{(\frac{\sum\limits_{i=1}^{n} (x_i-\overline{x})(y_i-\overline{y})}{n-1})^2}{\frac{\sum\limits_{i=1}^{n} (x_i-\overline{x})^2}{n-1}\frac{\sum\limits_{i=1}^{n}(y_i-\overline{y})^2}{n-1}}}=\sqrt{\frac{cov(x,y)^2}{var(x)var(y)}}
$$

If the correlation coefficient is close to 0, there is no correlation between both sets of samples. If it is close to 1, we have a direct correlation, if the value of the x sample goes up, the value of the y sample does too. If it is close to -1, we have an inverse correlation, if the sample of x goes up, the sample of y goes down.

To calculate the correlation coefficient between the amount of light and how much our plants grew, we still need to calculate the standard deviation of our hours of light. The variance is 3.5 and the square root of that is 1.87. Now we can calculate the correlation coefficient as 11.2/(1.87*6.03) which is 0.99.

Since our correlation coefficient is nearly 1, there is a high correlation between hours of light and plant growth.

## Covariance matrix

The covariance matrix is a matrix which contains all covariances between each two parameters. If we would build a covariance matrix between just two parameters x and y, like our light and plant growth  parameters, we would get the matrix

$$
\begin{bmatrix}cov(x, x) & cov(x, y) \\ cov(y, x) & cov(y,y)\end{bmatrix}
$$

Of course the covariance between two identical parameters is the variance, so we can write

$$
\begin{bmatrix}var(x) & cov(x, y) \\ cov(y, x) & var(y)\end{bmatrix}
$$

And since the covariance of x and y is equal to the covariance of y and x, the matrix is symmetric .

The more parameters we have, the larger the matrix will be as it needs to cover more combinations

$$
\begin{bmatrix}var(x) & cov(x, y) & cov(x, z) & ..\\ cov(y, x) & var(y) & cov(y,z) & ..\\cov(z,x) & cov(z,y) & var(z) & ..\\ .. & .. & .. & ..\end{bmatrix}
$$

We will shortly see why we are interested in this matrix, but before that we will recreate it using matrix math. While it is completely OK to create it using the methods we used before, and even more logical as we only need to calculate one of two symmetric triangles, matrix operations might be faster depending on the hardware used.

The steps are as follows. We start with an nxm matrix in which each each row defines a sample from 1 to n and each column defines a parameter from 1 to m

$$
\begin{bmatrix}x_1 & y_1 & z_1 & ..\\ x_2 & y_2 & z_2 & ..\\x_3 & y_3 & z_3 & ..\\ .. & .. & .. & ..\end{bmatrix}
$$

We multiply this matrix with a nxn matrix of ones

$$
\begin{bmatrix}1 & 1 & 1 & ..\\ 1 & 1 & 1 & ..\\1 & 1 & 1 & ..\\ .. & .. & .. & ..\end{bmatrix} * \begin{bmatrix}x_1 & y_1 & z_1 & ..\\ x_2 & y_2 & z_2 & ..\\x_3 & y_3 & z_3 & ..\\ .. & .. & .. & ..\end{bmatrix}=\begin{bmatrix}\sum\limits_{i=1}^{n}{x_i} & \sum\limits_{i=1}^{n}{y_i} & \sum\limits_{i=1}^{n}{z_i} & ..\\ \sum\limits_{i=1}^{n}{x_i} & \sum\limits_{i=1}^{n}{y_i} & \sum\limits_{i=1}^{n}{z_i} & ..\\\sum\limits_{i=1}^{n}{x_i} & \sum\limits_{i=1}^{n}{y_i} & \sum\limits_{i=1}^{n}{z_i} & ..\\ .. & .. & .. & ..\end{bmatrix}
$$

The result is that each column, from 1 to m will contain n rows, which all contain the sum of all sample values for that parameter. Then we divide by the amount of samples to get

$$
\begin{bmatrix}\sum\limits_{i=1}^{n}{x_i} & \sum\limits_{i=1}^{n}{y_i} & \sum\limits_{i=1}^{n}{z_i} & ..\\ \sum\limits_{i=1}^{n}{x_i} & \sum\limits_{i=1}^{n}{y_i} & \sum\limits_{i=1}^{n}{z_i} & ..\\\sum\limits_{i=1}^{n}{x_i} & \sum\limits_{i=1}^{n}{y_i} & \sum\limits_{i=1}^{n}{z_i} & ..\\ .. & .. & .. & ..\end{bmatrix}*\frac{1}{n}=\begin{bmatrix}\frac{\sum\limits_{i=1}^{n}{x_i}}{n} & \frac{\sum\limits_{i=1}^{n}{y_i}}{n} & \frac{\sum\limits_{i=1}^{n}{z_i}}{n} & ..\\ \frac{\sum\limits_{i=1}^{n}{x_i}}{n} & \frac{\sum\limits_{i=1}^{n}{y_i}}{n} & \frac{\sum\limits_{i=1}^{n}{z_i}}{n} & ..\\ \frac{\sum\limits_{i=1}^{n}{x_i}}{n} & \frac{\sum\limits_{i=1}^{n}{y_i}}{n} & \frac{\sum\limits_{i=1}^{n}{z_i}}{n} & ..\\ .. & .. & .. & ..\end{bmatrix}=\begin{bmatrix}\overline{x} & \overline{y} & \overline{z} & ..\\ \overline{x} & \overline{y} & \overline{z} & ..\\ \overline{x} & \overline{y} & \overline{z} & ..\\ .. & .. & .. & ..\end{bmatrix}
$$

This is a matrix which contains in every column for every row the mean of the samples. We subtract this from our original sample matrix to get the sample value minus the mean.

$$
\begin{bmatrix}x_1 & y_1 & z_1 & ..\\ x_2 & y_2 & z_2 & ..\\x_3 & y_3 & z_3 & ..\\ .. & .. & .. & ..\end{bmatrix}-\begin{bmatrix}\overline{x} & \overline{y} & \overline{z} & ..\\ \overline{x} & \overline{y} & \overline{z} & ..\\ \overline{x} & \overline{y} & \overline{z} & ..\\ .. & .. & .. & ..\end{bmatrix}=\begin{bmatrix}x_1-\overline{x} & y_1-\overline{y} & z_1-\overline{z} & ..\\ x_2-\overline{x} & y_2-\overline{y} & z_2-\overline{z} & ..\\x_3-\overline{x} & y_3-\overline{y} & z_3-\overline{z} & ..\\ .. & .. & .. & ..\end{bmatrix}
$$

To get the sum of squares, we multiply the transpose of this matrix with itself

$$
\begin{bmatrix}x_1-\overline{x} & x_2-\overline{x} & x_3-\overline{x} & ..\\ y_1-\overline{y} & y_2-\overline{y} & y_3-\overline{y} & ..\\z_1-\overline{z} & z_2-\overline{z} & z_3-\overline{z} & ..\\ .. & .. & .. & ..\end{bmatrix}*\begin{bmatrix}x_1-\overline{x} & y_1-\overline{y} & z_1-\overline{z} & ..\\ x_2-\overline{x} & y_2-\overline{y} & z_2-\overline{z} & ..\\x_3-\overline{x} & y_3-\overline{y} & z_3-\overline{z} & ..\\ .. & .. & .. & ..\end{bmatrix}
$$

This gives us

$$
\begin{bmatrix}\sum\limits_{i=1}^{n}{(x_i-\overline{x})^2} & \sum\limits_{i=1}^{n}{(x_i-\overline{x})(y_i-\overline{y})} & \sum\limits_{i=1}^{n}{(x_i-\overline{x})(z_i-\overline{z})} & ..\\ \sum\limits_{i=1}^{n}{(y_i-\overline{y})(x_i-\overline{x})} & \sum\limits_{i=1}^{n}{(y_i-\overline{y})^2} & \sum\limits_{i=1}^{n}{(y_i-\overline{y})(z_i-\overline{z})} & ..\\ \sum\limits_{i=1}^{n}{(z_i-\overline{z})(x_i-\overline{x})} & \sum\limits_{i=1}^{n}{(z_i-\overline{z})(y_i-\overline{y})} & \sum\limits_{i=1}^{n}{(z_i-\overline{z})^2} & ..\\ .. & .. & .. & ..\end{bmatrix}
$$

Which we can divide by the amount of samples minus 1 to get

$$
\begin{bmatrix}\frac{\sum\limits_{i=1}^{n}{(x_i-\overline{x})^2}}{n-1} & \frac{\sum\limits_{i=1}^{n}{(x_i-\overline{x})(y_i-\overline{y})}}{n-1} & \frac{\sum\limits_{i=1}^{n}{(x_i-\overline{x})(z_i-\overline{z})}}{n-1} & ..\\ \frac{\sum\limits_{i=1}^{n}{(y_i-\overline{y})(x_i-\overline{x})}}{n-1} & \frac{\sum\limits_{i=1}^{n}{(y_i-\overline{y})^2}}{n-1} & \frac{\sum\limits_{i=1}^{n}{(y_i-\overline{y})(z_i-\overline{z})}}{n-1} & ..\\ \frac{\sum\limits_{i=1}^{n}{(z_i-\overline{z})(x_i-\overline{x})}}{n-1} & \frac{\sum\limits_{i=1}^{n}{(z_i-\overline{z})(y_i-\overline{y})}}{n-1} & \frac{\sum\limits_{i=1}^{n}{(z_i-\overline{z})^2}}{n-1} & ..\\ .. & .. & .. & ..\end{bmatrix}
$$

Which is the covariance matrix.

On the diagonal we can see how much every parameter varies among samples, and the off-diagonal elements show how much relationship there is between parameters.

## Principal Component Analysis or PCA

When we have many parameters per sample, we can find which parameters or parameter groups cause the most change by using principal component analysis.

To do a principal component analysis we take the covariance matrix. For this matrix we search for the eigenvectors and the corresponding eigenvalues. The eigenvectors with the largest eigenvalues indicate which parameters work together to influence the most change.

Eigenvectors are vectors which don't change direction when transformed by the matrix. Eigenvalues are the scale factors by which the eigenvectors are scaled when transformed by the matrix.

[More content coming soon]

## Deriving the regression formulas

Let's show that both formulas for linear regression are identical. We start with the formula for covariance of x and y over the variance of x

$$
\frac{cov(x, y)}{var(x)}
$$

or

$$
a=\frac{\frac{\sum\limits_{i=1}^{n} (x_i-\overline{x})(y_i-\overline{y})}{n-1}}{\frac{\sum\limits_{i=1}^{n} (x_i-\overline{x})^2}{n-1}}
$$

Since the $$n-1$$ cancel eachother out we get

$$
a=\frac{\sum\limits_{i=1}^{n} (x_i-\overline{x})(y_i-\overline{y})}{\sum\limits_{i=1}^{n} (x_i-\overline{x})^2}
$$

Then we write out the product

$$
a=\frac{\sum\limits_{i=1}^{n} (x_iy_i-x_i\overline{y}-y_i\overline{x}+\overline{x}\overline{y})}{\sum\limits_{i=1}^{n} (x_ix_i-x_i\overline{x}-x_i\overline{x}+\overline{x}\overline{x})}
$$

Now we can split the terms

$$
a=\frac{\sum\limits_{i=1}^{n}x_iy_i-\sum\limits_{i=1}^{n}x_i\overline{y}-\sum\limits_{i=1}^{n}y_i\overline{x}+\sum\limits_{i=1}^{n}\overline{x}\overline{y}}{\sum\limits_{i=1}^{n}x_ix_i-\sum\limits_{i=1}^{n}x_i\overline{x}-\sum\limits_{i=1}^{n}x_i\overline{x}+\sum\limits_{i=1}^{n}\overline{x}\overline{x}}
$$

We can pull out factors independent of i

$$
a=\frac{\sum\limits_{i=1}^{n}x_iy_i-\overline{y}\sum\limits_{i=1}^{n}x_i-\overline{x}\sum\limits_{i=1}^{n}y_i+N\overline{x}\overline{y}}{\sum\limits_{i=1}^{n}x_ix_i-\overline{x}\sum\limits_{i=1}^{n}x_i-\overline{x}\sum\limits_{i=1}^{n}x_i+N\overline{x}\overline{x}}
$$

Since $$\overline{x}=\frac{\sum\limits_{i=1}^{n}x_i}{n}$$ and $$\overline{y}=\frac{\sum\limits_{i=1}^{n}y_i}{n}$$

$$
a=\frac{\sum\limits_{i=1}^{n}x_iy_i-\frac{\sum\limits_{i=1}^{n}y_i}{n}\sum\limits_{i=1}^{n}x_i-\frac{\sum\limits_{i=1}^{n}x_i}{n}\sum\limits_{i=1}^{n}y_i+n\frac{\sum\limits_{i=1}^{n}x_i}{n}\frac{\sum\limits_{i=1}^{n}y_i}{n}}{\sum\limits_{i=1}^{n}x_ix_i-\frac{\sum\limits_{i=1}^{n}x_i}{n}\sum\limits_{i=1}^{n}x_i-\frac{\sum\limits_{i=1}^{n}x_i}{n}\sum\limits_{i=1}^{n}x_i+n\frac{\sum\limits_{i=1}^{n}x_i}{n}\frac{\sum\limits_{i=1}^{n}x_i}{n}}
$$

Grouping terms gives

$$
a=\frac{\sum\limits_{i=1}^{n}x_iy_i-2\frac{\sum\limits_{i=1}^{n}y_i\sum\limits_{i=1}^{n}x_i}{n}+\frac{\sum\limits_{i=1}^{n}x_i\sum\limits_{i=1}^{n}y_i}{n}}{\sum\limits_{i=1}^{n}x_ix_i-2\frac{\sum\limits_{i=1}^{n}x_i\sum\limits_{i=1}^{n}x_i}{n}+\frac{\sum\limits_{i=1}^{n}x_i\sum\limits_{i=1}^{n}x_i}{n}}
$$

Further grouping gives

$$
a=\frac{\sum\limits_{i=1}^{n}x_iy_i-\frac{\sum\limits_{i=1}^{n}y_i\sum\limits_{i=1}^{n}x_i}{n}}{\sum\limits_{i=1}^{n}x_ix_i-\frac{\sum\limits_{i=1}^{n}x_i\sum\limits_{i=1}^{n}x_i}{n}}
$$

Multiplying both numerator and denominator by n gives

$$
a=\frac{n\sum\limits_{i=1}^{n}x_iy_i-\sum\limits_{i=1}^{n}y_i\sum\limits_{i=1}^{n}x_i}{n\sum\limits_{i=1}^{n}x_ix_i-\sum\limits_{i=1}^{n}x_i\sum\limits_{i=1}^{n}x_i}
$$

Which is the formula we were aiming for.