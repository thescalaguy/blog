---
title: Precision of OLS Estimates
tags:
  - machine learning
date: 2019-03-26 15:41:57
---


The calculation of the estimators $\hat{\beta}_1$ and $\hat{\beta}_2$ is based on sample data. As the sample drawn changes, the value of these estimators also changes. This leaves us with the question of how reliable these estimates are i.e. we'd like to determine the **precision** of these estimators. This can be determined by calculating the **standard error** or **the coefficient of determination**.  

## Standard Error  

The standard error of a statistic<sup>[1]</sup> is the standard deviation of the sampling distribution of that statistic.   

Suppose we have a dataset which contains incomes of people. From this dataset we start drawing samples of size `n` and calculating the mean. Now if we plot the distribution (the sampling distribution)  of the means we calculated, we'll get a normal distribution centered on the population mean. The standard deviation of this sampling distribution is the standard error of the statistic which in our case is mean. 

Standard error is given by the formula: $$\sigma_{\bar{x}} = \frac{\sigma}{\sqrt{n}}$$ 

Where   
$\sigma$ is the standard deviation of the population. 
$n$ is the sample size.

Calculating the standard error shows the sampling fluctuation. Sampling fluctuation shows the extent to which a statistic takes on different values.  

Notice that the standard error has an inverse relation with the sample size `n`. This means that the larger the sample we draw from the population, the lesser the standard error will be. This will also result in a tighter normal distribution since the standard deviation will be less.  

## Standard Error of OLS Estimates  

The standard error of the OLS estimators $\hat{\beta}_1$ and $\hat{\beta}_2$ is given by: 

$$
\begin{align}
se(\hat{\beta}_2) &= \frac{\sigma}{\sqrt{\sum{x_i^2}}} \\\\ 
se(\hat{\beta}_1) &= \sqrt{\frac{\sum{X_i^2}}{n\sum{x_i^2}}}\sigma
\end{align}
$$

where $\sigma$ is the square root of true but unknown constant of homoscedastic variance $\sigma^2$.  

All of the terms in the equations above except $\sigma^2$ can be calculated from the sample drawn. Therefore, we will need an unbiased estimator $\hat{\sigma}^2 = \frac{\sum{\hat{u}_i^2}}{n - 2}$ . The denominator $n-2$ represents the degrees of freedom. $\sum{\hat{u}_i^2}$ is the residual sum of squares. 

Although $\hat{u}_i = Y_i - \hat{Y}_i$, its summation can be computed with an alternative formula as $$\sum{\hat{u}_i^2} = \sum{y_i^2} - \frac{(\sum{x_iy_i})^2}{\sum{x_i^2}}$$.   

All of the terms in the above formula would have already been computed as a part of computing the estimators.  

## How it all ties together  

As you calculate the estimators and draw the sample regression curve that passes through your data, you need some numeric measure of how good the curve fits the data i.e. a measure of "goodness of fit". This can be given as:   

$$ \hat{\sigma} = \sqrt{\frac{\sum{\hat{u}_i^2}}{n-2}}  $$  

This is the positive square root of the estimator of homoscedastic variance. This is the standard deviation of the $Y$ values about the regerssion curve.  

The standard errors of the estimators $\hat{\beta}_1$ and $\hat{\beta}_2$ will show you how much they fluctuate as you draw the samples. The lesser their standard error, the better. 

The square of the standard error is called the mean squared error.

## Let's see some code  

To start off, let's load the [Boston housing](https://www.cs.toronto.edu/~delve/data/boston/bostonDetail.html) dataset.

{% codeblock lang:python %}
import pandas as pd
from sklearn import datasets
boston = datasets.load_boston()
data, target = boston.data, boston.target
df = pd.DataFrame(data=data, columns=boston.feature_names)
df = df[['RM']]
df['RM'] = df['RM'].apply(round)
df['price'] = target 
{% endcodeblock %}

I'm choosing to use the average number of rooms as the variable to see how it affects price. I've rounded it off to make the calculations easier. The scatter plot of the data looks the following:  

{% asset_img boston_scatter.png [Population Scatter Plot] %}

This, ofcourse, is the entire population data so let's draw a sample.  

{% codeblock lang:python %}
sample = df.sample(n=100)
{% endcodeblock %}

Now let's bring back our function that calculates the OLS estimates:  

{% codeblock lang:python %}
def ols(sample):
    Xbar = sample['X'].mean()
    Ybar = sample['Y'].mean()

    sample['x'] = sample['X'] - Xbar
    sample['x_sq'] = sample['x'] ** 2
    sample['y'] = sample['Y'] - Ybar
    sample['xy'] = sample['x'] * sample['y']

    beta2cap = sample['xy'].sum() / sample['x_sq'].sum()
    beta1cap = Ybar - (beta2cap * Xbar)

    sample['Ycap'] = beta1cap + beta2cap * sample['X']
    sample['ucap'] = sample['Y'] - sample['Ycap']

    return beta1cap, beta2cap
{% endcodeblock %}

and now let's see coefficients

{% codeblock lang:python %}
sample.rename(columns={'RM': 'X', 'price': 'Y'}, inplace=True)
intercept, slope = ols(sample)
{% endcodeblock %} 

The plot of the regression curve on the sample data looks the following:  

{% asset_img regression_curve.png [Regression Curve] %}

The `sample` DataFrame has our intermediate calculation and we can use that to calculate the standard error. Let's write a function which does that.  

{% codeblock lang:python %}
from math import sqrt
def standard_error(sample):
    return sqrt((sample['ucap'] ** 2).sum() / (len(sample) - 2))
{% endcodeblock %}

Finally, let's see how much standard deviation we have around the regression line.  

{% codeblock lang:python %}
standard_error(sample)
7.401174774558201
{% endcodeblock %}

## Coefficient of Determination  

The coefficient of determination denoted by $r^2$ (for bivariate regression) or $R^2$ (for multivariate regression) is a ratio of the explained variance to the total variance of the data. The higher the coefficient of determination, the more accurately the regressors (average number of rooms in the house) explain the regressand (the price of the house) i.e. the better your regression model is. 

For bivariate linear regression, $r^2$ value is given by:  

$$ 
r^2 = \frac{\sum{(\hat{Y}_i - \bar{Y})^2}}{\sum{(Y_i - \bar{Y})^2}}
$$ 

$r^2$ can take on a value between 0 and 1.

Turning this into code, we have:  

{% codeblock lang:python %}
def coeff_of_determination(sample):
    Ybar = sample['Y'].mean()
    return ((sample['Ycap'] - Ybar) ** 2).sum() / ((sample['Y'] - Ybar) ** 2).sum()
{% endcodeblock %}

For the sample we've drawn, the $r^2$ value comes out to be `0.327`. This means that only 32.7% of the variance in the total data is explained by the regression model.

Finito.

<hr /> 

[1] A "statistic" is a numerical quantity such as mean, or median. 

