---
layout: single
title:  "Introduction to statistics"
date:   2018-05-11 22:03:01 +0900
categories: math
---

## Data

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
\overline{x}=\frac{\sum_{i=1}^{n} x_i}{n}
$$

In case we only have samples, rather than a population, we need to divide by $$n-1$$ instead of $$n$$.

$$
\overline{x}=\frac{\sum_{i=1}^{n} x_i}{n-1}
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
\frac{\sum_{i=1}^{n} (x_i-\overline{x})^2}{n-1}
$$

Thus it is the sample mean of the squared differences of each sample $$x$$ with the mean $$\overline{x}$$.

Conceptually, the variance is the average squared distance of a sample to the mean.

To calculate the variance of the plant growth we take all differences -7, -6, -2, 2, 5, 8 and square them to obtain 49, 36, 4, 4, 25, 64. We add all of these together and divide them by 5 to get (49+36+4+4+25+64)/5 or 36.4.

## Standard deviation

If we take the square root of the variance, we get the standard deviation. Since we take the square root of the squared average distance to the mean, this is the average distance of a sample to the mean.

$$
\sqrt{\frac{\sum_{i=1}^{n} (x_i-\overline{x})^2}{n-1}}
$$

The square root of the variance of our plant growth is $$\sqrt{36.4}$$ or 6.03, about 6. Since our values of growth go from 0 to 15, a standard deviation of 6 means there is quite some difference between our plant growth samples. It means that the average distance from the mean growth of 7mm is 6mm.

## Covariance

Until now, we worked with one measurement, $$x$$. If we want to compare an input parameter with our measurement, we use the covariance. The covariance is calculated by taking the sum of the products of the differences of each sample with its mean, divided by n-1. We basically swapped one $$(x_i-\overline{x})$$ with $$(y_i-\overline{y})$$.

$$
\frac{\sum_{i=1}^{n} (x_i-\overline{x})(y_i-\overline{y})}{n-1}
$$

Th covariance when calculated for the amount of light and plant growth is 11.2. To calculate this, we calculated the mean of our hours of light, which is 2.5. Then we took the sum of all products of the differences between light and light mean and growth and growth mean, finally dividing by 5.

## R

The correlation coefficient is to covariance as the standard deviation is to variance. It is a better measure to interpret results from.
The correlation coefficient is calculated by dividing the covariance by the product of the standard deviations of each measurement.

$$
\frac{\frac{\sum_{i=1}^{n} (x_i-\overline{x})(y_i-\overline{y})}{n-1}}{\sqrt{\frac{\sum_{i=1}^{n} (x_i-\overline{x})^2}{n-1}}\sqrt{\frac{\sum_{i=1}^{n} (y_i-\overline{y})^2}{n-1}}}
$$

If the correlation coefficient is close to 0, there is no correlation between both sets of samples. If it is close to 1, we have a direct correlation, if the value of the x sample goes up, the value of the y sample does too. If it is close to -1, we have an inverse correlation, if the sample of x goes up, the sample of y goes down.

To calculate the correlation coefficient between the amount of light and how much our plants grew, we still need to calculate the standard deviation of our hours of light. The variance is 3.5 and the square root of that is 1.87. Now we can calculate the correlation coefficient as 11.2/(1.87*6.03) which is 0.99.

Since our correlation coefficient is nearly 1, there is a high correlation between hours of light and plant growth.

## Linear regression

When we have two sets of samples, we can calculate a line which follows the trend of the samples. This line visually illustrates the correlation coefficient and can be used to extrapolate or guess values for which we don't have samples.

A line can be defined as an equation $$y=ax+b$$, where a is the slope and b is the position where the line crosses the x axis, thus where y is 0.

We can calculate the slope a as the covariance of x and y divided by the variance of x or

$$
a=\frac{\frac{\sum_{i=1}^{n} (x_i-\overline{x})(y_i-\overline{y})}{n-1}}{\frac{\sum_{i=1}^{n} (x_i-\overline{x})^2}{n-1}}
$$

Since the $$N-1$$ cancel eachother out we get

$$
a=\frac{\sum_{i=1}^{n} (x_i-\overline{x})(y_i-\overline{y})}{\sum_{i=1}^{n} (x_i-\overline{x})^2}
$$

Then we write out the product

$$
a=\frac{\sum_{i=1}^{n} (x_iy_i-x_i\overline{y}-y_i\overline{x}+\overline{x}\overline{y})}{\sum_{i=1}^{n} (x_ix_i-x_i\overline{x}-x_i\overline{x}+\overline{x}\overline{x})}
$$

Now we can split the terms

$$
a=\frac{\sum_{i=1}^{n}x_iy_i-\sum_{i=1}^{n}x_i\overline{y}-\sum_{i=1}^{n}y_i\overline{x}+\sum_{i=1}^{n}\overline{x}\overline{y}}{\sum_{i=1}^{n}x_ix_i-\sum_{i=1}^{n}x_i\overline{x}-\sum_{i=1}^{n}x_i\overline{x}+\sum_{i=1}^{n}\overline{x}\overline{x}}
$$

We can pull out factors not independent of i

$$
a=\frac{\sum_{i=1}^{n}x_iy_i-\overline{y}\sum_{i=1}^{n}x_i-\overline{x}\sum_{i=1}^{n}y_i+N\overline{x}\overline{y}}{\sum_{i=1}^{n}x_ix_i-\overline{x}\sum_{i=1}^{n}x_i-\overline{x}\sum_{i=1}^{n}x_i+N\overline{x}\overline{x}}
$$

Since $$\overline{x}=\frac{\sum_{i=1}^{n}x_i}{N}$$ and $$\overline{y}=\frac{\sum_{i=1}^{n}y_i}{N}$$

$$
a=\frac{\sum_{i=1}^{n}x_iy_i-\frac{\sum_{i=1}^{n}y_i}{N}\sum_{i=1}^{n}x_i-\frac{\sum_{i=1}^{n}x_i}{N}\sum_{i=1}^{n}y_i+N\frac{\sum_{i=1}^{n}x_i}{N}\frac{\sum_{i=1}^{n}y_i}{N}}{\sum_{i=1}^{n}x_ix_i-\frac{\sum_{i=1}^{n}x_i}{N}\sum_{i=1}^{n}x_i-\frac{\sum_{i=1}^{n}x_i}{N}\sum_{i=1}^{n}x_i+N\frac{\sum_{i=1}^{n}x_i}{N}\frac{\sum_{i=1}^{n}x_i}{N}}
$$

Combining terms gives

$$
a=\frac{\sum_{i=1}^{n}x_iy_i-2\frac{\sum_{i=1}^{n}y_i\sum_{i=1}^{n}x_i}{N}+\frac{\sum_{i=1}^{n}x_i\sum_{i=1}^{n}y_i}{N}}{\sum_{i=1}^{n}x_ix_i-2\sum_{i=1}^{n}x_i\sum_{i=1}^{n}x_i-\sum_{i=1}^{n}x_i\sum_{i=1}^{n}x_i+\frac{\sum_{i=1}^{n}x_i\sum_{i=1}^{n}x_i}{N}}
$$

Somehow we get to

$$
a=\frac{\frac{\sum{xy}}{N}-\frac{\sum{x}}{N}\frac{\sum{y}}{N}}{\frac{\sum{x^2}}{N}-(\frac{\sum{x}}{N})^2}
$$

If we multiply the first terms with $$\frac{N}{N}$$ we get

$$
a=\frac{\frac{N\sum{xy}}{N^2}-\frac{\sum{x}\sum{y}}{N^2}}{\frac{N\sum{x^2}}{N^2}-\frac{(\sum{x})^2}{N^2}}
$$

Now we see that the $$N^2$$ denominators cancel eachother out and come to the following formula to calculate a

$$
a=\frac{N\sum{xy}-\sum{x}\sum{y}}{N\sum{x^2}-(\sum{x})^2}
$$

Once we have a, we can calculate b since

$$
\frac{\sum{y}}{N}=a\frac{\sum{x}}{N}+b
$$

We can subtract $$a\frac{\sum{x}}{N}$$ from both sides to get

$$
b=\frac{\sum{y}}{N}-a\frac{\sum{x}}{N}
$$

or if we group the fractions

$$
b=\frac{\sum{y}-a\sum{x}}{N}
$$