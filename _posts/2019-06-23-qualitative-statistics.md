---
layout: single
title:  "Introduction to statistics: Part 2"
date:   2019-06-23 12:27:01 +0900
categories: math
excerpt: "A short introduction to statistical methods for qualitative data."
toc: true
---

## Nonparametric and qualitative data

In a lot of cases, we don't only have numbers, but categories as well. This can happen when we place measurable values into buckets, like [low, medium, high] or when we have distinct possibilities which are not necessarily ordered, like [red, green, blue, yellow]. These are qualitative parameters, while in part 1 we only worked with quantitative parameters.

### Spearman's rank correlation coefficient

When the data we use is nonparametric, but still has a certain order, we can use Spearman's rank correlation coefficient instead of the Pearson correlation coefficient.

#### Calculating the rank

The Spearman's rank correlation coefficient is obtained by calculating the rank of every value, and then using these values to calculate the Pearson correlation coefficient.

There are several methods to calculate the rank of a list of values. The simplest method is to sort the values in ascending or descending order and to assign the values numbers from 1 until n, the amount of values. However if a value appears more than once, all these duplicates are assigned the rank of the first duplicate, while the numbering keeps going up silently, so when a new unique value is encountered, the numbering jumps.

| Value        | Rank        |
| -------------| ----------- |
| 2            | 1           |
| 3            | 2           |
| 4            | 3           |
| 4            | 3           |
| 4            | 3           |
| 5            | 6           |

Every value 4 receives rank 3. Rank 4 and 5 are skipped here, while value 5 receives the next rank, rank 6.

Alternatively the duplicates can be assigned the average of their lowest and highest rank.

| Value        | Rank        |
| -------------| ----------- |
| 2            | 1           |
| 3            | 2           |
| 4            | 4           |
| 4            | 4           |
| 4            | 4           |
| 5            | 6           |

The values 4 would have received rank 3, 4 and 5. The average of 3 and 5 is 4, so all values receive rank 4.

#### Calculating the correlation coefficient

Now that we can calculate the rank of each element in a list of values, we can calculate the Spearman's rank correlation coefficient as follows

$$
\frac{cov(x',y')}{std(x')std(y')}
$$

Where $$x'$$ and $$y'$$ are the list of values obtained by replacing each value in $$x$$ and $$y'$$ by its rank.

#### Significance of the correlation coefficient

Just having the coefficient isn't enough to make a conclusion. To know how significant the result is, which depends on how much samples we have, we need to calculate the probability. To do this, we first calculate the tscore as

$$t=\frac{r*\sqrt{n-2}}{\sqrt{1-r^2}}$$

Then we can calculate the probability using TDIST(t, n-2, 2) in Excel or pt(-abs(x), n-2) * 2 in R. The lower the probability, the more unusual our result is. Usually a cutoff value of 0.05 is used. If our probability is lower than 0.05, we say that there is a correlation between the parameters, as there is a very small chance that our samples come from a distribution where the parameters are unrelated.

### Cramer's V

This method can be used when both our data series are using categories rather than numbers.

To calculate Cramer's V, we first calculate the frequency of each pair. If series x has categories from 1 to k, and series y has categories from 1 to r, then $$n_{ij}$$ holds the count of all data pairs where x is i and y is j. $$n_i$$ is the amount of data pairs where x is i, and $$n_j$$ is the amount of data pairs where y is j. $$n$$ itself is the amount of samples.

Then we calculate $$\chi^{2}$$ or Chi squared from these frequencies

$$\chi^2=\sum_{i,j}{\frac{(n_{ij}-\frac{n_{i}n_{j}}{n})^2}{\frac{n_{i}n_{j}}{n}}}$$

And we can calculate V from $$\chi^{2}$$ as follows

$$V=\sqrt{\frac{\chi^2/n}{\min(k-1,r-1)}}$$

Where 0 means that the parameters are independent, and 1 that the parameters are dependent. So the value of V tells us how much the two parameters are related.

What exactly are we calculating though? Let's take a step back, and use an example. Let's say we have a list of records with people's gender and hair color. We can create a table listing the frequency of each combination

|              | Black | Brown | Red | Blond |
| ------------ | ----- | ----- |---- |------ |
| Male         | 56    | 143   | 34  | 46    |
| Female       | 52    | 143   | 37  | 81    |

So 56 of the people in our set have black hair and are male.

Our table now contains the $$n_{ij}$$ values, from which we can calculate $$n_i$$ (108, 286, 71, 127), $$n_j$$ (279, 313) and $$n$$ (592)

|              | Black | Brown | Red | Blond | Sum |
| ------------ | ----- | ----- |---- |------ | --- |
| Male         | 56    | 143   | 34  | 46    | 279 |
| Female       | 52    | 143   | 37  | 81    | 313 |
| Sum          | 108   | 286   | 71  | 127   | 592 |

Now we create a table which contains $$\frac{n_{i}n_{j}}{n}$$ for each i and j

|              | Black       | Brown       | Red         | Blond       |
| ------------ | ----------- | ----------- |------------ |------------ |
| Male         | 50.89864865 | 134.7871622 | 33.46114865 | 59.85304054 |
| Female       | 57.10135135 | 151.2128378 | 37.53885135 | 67.14695946 |

This table contains what we call expected values. It is the table you would get if the parameters gender and hair color were completely unrelated. For example males with black hair. we know there are 108 people with black hair, and 279 people of 592 are male. This means that in a perfect case of unrelated parameters 108*279/592 or 50.89864865 would be males with black hair.

If we replace $$\frac{n_{i}n_{j}}{n}$$ with $$E_{ij}$$ or the expected value at ij, we can rewrite the formula for $$\chi^2$$ as

$$\chi^2=\sum_{i,j}{\frac{(n_{ij}-E_{ij})^2}{E_{ij}}}$$

So we are looking at the scaled squared distance between the observed and expected values, and summing these scaled distances.

So the $$\chi^2$$ value tells us how much the two sample distributions are apart. In our case $$\chi^2$$ is 7.994244189.

This is actually what CHITEST(observed, expected) uses in Excel. It calculates $$\chi^2$$ like we did and calculates the degrees of freedom df

$$df=max(1, r-1)*max(1, c-1)$$

Then it calculates the probability. We can do this using CHIDIST($$\chi^2$$, df) in Excel or pchisq($$\chi^2$$, df, lower.tail=FALSE) in R. In our case p is 0.04613081084.

If the probability is beneath a threshold, 0.05 for example, it means that the probability that the samples are from the same population is lower than 5%, so our observed values are probably not from a population where the categories are unrelated, thus there might be a relation between the two parameters. Since 0.04613081084 is slightly less than 0.05, we might say that there is more than 0 correlation between gender and hair color. How much though, is hard to tell.

Cramer’s V however, gives an easier to interpret value between 0 and 1, without the need to choose a threshold. In our case V is 0.1162058125, which means there is a slight correlation between gender and hair color.

In R, cramersV(x) gives V from the table x.

### The Uncertainty Coefficient

Cramer's V gives us the same value irrespective of the order in we pass our parameters. So we can't see the direction of a possible relation. However it might be that it is easier to guess whether a person is female when we know the hair color is blond than knowing the hair color when we know the person is female, V won't provide this information.

The uncertainty coefficient will tell us how certain it is that we can guess x given y.

To calculate this, we need a few other concepts first. The uncertainty coefficient is based on information theory and the concept of entropy.

The entropy of a list of data can be calculated as

$$H(X)=-\sum_{i=1}^{n}{P(x_{i})\log_{e}P(x_{i})}$$

With $$P(x_{i})$$ the probability of $$x_{i}$$ occurring.

The higher the entropy, the more uncertainty or surprise there is. This happens when some probabilities are small. This is because the smaller the probability, the larger the logarithm of the probability is. Since we use $$log_e$$, our probability of 0 to 1 maps onto -∞ to 0. So the - sign makes sure the entropy is positive.

When we have events involving the probability that one parameter has a certain value and another parameter has a certain value, we can calculate the conditional entropy.

The entropy of the data pairs with parameters X and Y can be written as

$$H(X,Y)=-\sum_{i=1,j=1}^{n}{P(x_{i},y_{j})\log_{e}{P(x_{i},y_{j})}}$$

With $$P(x_{i},y_{j})$$ the probability of $$x_{i}$$ and $$y_{i}$$ occurring simultaneously.

Now we can calculate conditional entropy. If we subtract the entropy of Y from H(X, Y), because we know Y, we get

$$H(X|Y)=H(X,Y)-H(Y)$$

or

$$H(X|Y)=-\sum_{i=1,j=1}^{n}{P(x_{i},y_{j})\log_{e}{P(x_{i},y_{j})}}+\sum_{i=1}^{n}{P(y_{i})\log_{e}{P(y_{i})}}$$

Which gives us

$$H(X|Y)=\sum_{i,j}P(x_{i},y_{j})\log_{e}{\frac{P(y_{j})}{P(x_{i},y_{j})}}$$

This tells us how much uncertainty remains in X when we know Y.

Given these definitions, we can now calculate the uncertainty coefficient as

$$U(X|Y)=\frac{H(X)-H(X|Y)}{H(X)}$$

This gives us a number between 0 and 1 which is a measure of how good we can predict X from Y.

To use this on our example, we first calculate the frequency lists like we did with Cramer's V. For X we get

| Black       | Brown       | Red         | Blond       |
| ----------- | ----------- |------------ |------------ |
| 108         | 286         | 71          | 127         |

And for Y

| Male        | Female      |
| ----------- | ----------- |
| 279         | 313         |

We can calculate probabilities from these by dividing by the amount of samples, 592.

| Black       | Brown       | Red         | Blond       |
| ----------- | ----------- |------------ |------------ |
| 108         | 286         | 71          | 127         |
| 0.1824324324| 0.4831081081| 0.1199324324| 0.214527027 |

| Male        | Female      |
| ----------- | ----------- |
| 279         | 313         |
| 0.4712837838| 0.5287162162|

Now we can calculate the entropy H(X) = 1.2464359225967288, as well as H(Y) = 0.6914970305474704, which give the entropy of X and Y respectively.

Next we calculate the conditional entropy of H(X\|Y). For this we need our pair data

|              | Black | Brown | Red | Blond |
| ------------ | ----- | ----- |---- |------ |
| Male         | 56    | 143   | 34  | 46    |
| Female       | 52    | 143   | 37  | 81    |

We calculate probabilities for these by dividing by our sample count, 592.

|              | Black         | Brown        | Red           | Blond        |
| ------------ | ------------- | ------------ |-------------- |------------- |
| Male         | 0.09459459459 | 0.2415540541 | 0.05743243243 | 0.0777027027 |
| Female       | 0.08783783784 | 0.2415540541 | 0.0625        | 0.1368243243 |

And calculate H(X\|Y) which gives 1.239600755.

Finally we calculate the uncertainty coefficient U(X\|Y) which is 0.00548376956. This measures how good we can guess the hair color given that we know the gender. Not null, but not very high, so the correlation is there, but very low.

We can also calculate U(Y\|X) to see if the reverse is really different. U(Y\|X) is 0.009884593959. This measures how good we can guess the gender given that we know the hair color. Also not very high, but double of U(X\|Y), which means that it is slightly easier to guess the gender than the hair color.

In R, given a table x, the uncertainty coefficient can be found using nom.uncertainty(x).

There is also a symmetric form of the uncertainty coefficient

$$U(X,Y)=2\frac{H(X)+H(Y)-H(X,Y)}{H(X)+H(Y)}$$

Which, like Cramer's V is not dependent on the order of the parameters, so

$$U(X,Y)=U(Y,X)$$

While this is not necessarily true for U(X\|Y) and U(Y\|X).

In our case H(X,Y) is 0.003527040169, which again points to a very weak correlation.

### Correlation Ratio

When we have one qualitative parameter and one quantitative parameter, we can use the correlation ratio to look for correlation.

We start by sorting all the quantitative values into buckets, one for each possible value of the qualitative parameter. So if our qualitative parameter is gender, and our quantitative parameter is the salary, we make two lists, one for each gender, with all the salaries for that particular gender.

Then we calculate the average for each list, which is the average salary by gender, $$\overline{y_i}$$. We also calculate the average of all salaries, $$\overline{y}$$.

Now we can calculate the correlation ratio as

$$\frac{\sum_i{n_i(\overline{y_i}-\overline{y})^2}}{\sum_i{(y-\overline{y})^2}}$$

We can see that the numerator gives us the deviation of each bucket’s average to the total average, scaled by the amount of elements in the bucket. While the denominator gives us the deviation of all values to the total average. What does this mean? If one or more buckets have a disproportionate amount of larger numbers, the average of those buckets will be greater, and thus the deviation to the total average will be greater. This happens when there is a correlation, as in the case of a correlation, the bucket a value is in will influence its magnitude.
