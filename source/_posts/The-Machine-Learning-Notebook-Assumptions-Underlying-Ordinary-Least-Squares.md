---
title: Assumptions Underlying Ordinary Least Squares
tags:
  - machine learning
date: 2018-11-12 09:02:15
mathjax: true
---


In the previous post we calculated the values of $\hat{\beta}_1$ and $\hat{\beta}_2$ from a given sample and in closing I mentioned that we'll take a look at finding out whether the sample regression line we obtained is a good fit. In other words, we need to assess how close $\hat{\beta}_1$ and $\hat{\beta}_2$ are to $\beta_1$ and $\beta_2$ or how close $\hat{Y}_i$ is to $E(Y|X_i)$. Recall that the population regression function (PRF) is given by $Y_i = \beta_1 + \beta_2X_i + u_i$. This shows that the value of $Y_i$ depends on $X_i$ and $u_i$. To be able to make any statistical inference about $Y_i$ (and $\beta_1$ and $\beta_2$) we need to make assumptions about how the values of $X_i$ and $u_i$ were generated. We'll discuss these assumptions in this post as they're important for the valid interpretation of the regression estimates. 

## Classic Linear Regression Model (CLRM)  

There are 7 assumptions that **Gaussian** or **Classic** linear regression model (CLRM) makes. These are:  

### 1. The regression model is linear in parameters  

Recall that the population regression function is given by $Y_i = \beta_1 + \beta_2X_i + u_i$. This is linear in parameters since $\beta_1$ and $\beta_2$ have power 1. The explanatory variables, however, do not have to be linear.  

### 2. Values of X are fixed in repeated sampling  

In this series of posts, we'll be looking at **fixed regressor models**<sup>[1]</sup> where the values of X are considered to be fixed when drawing a random sample. For example, in the previous post the values of X were fixed between 80 and 260. If another sample were to be drawn, the sample values of X would be used.  

### 3. Zero mean value of the error term $u_i$  

This assumption states that for every given $X_i$, the error term $u_i$ is zero, on average. This should be intuitive. The regression line passes through the conditional mean for every $X_i$. The error term $u_i$ are distances of the individual points $Y_i$ from the conditional mean. Some of them lie above the regression line and some of them lie below. However, they cancel each other out; the positive $u_i$ cancel out the negative $u_i$ and therefore $u_i$ is zero, on average. Notationally, $E(u_i|X_i) = 0$. 

What this assumption means that all those variables that were not considered to be a part of the regression model do not systematically affect $E(Y|X_i)$ i.e. their effect on average is zero on $E(Y|X_i)$. 

This assumption also means is that there is no **specification error** or **specification bias**. This happens when we chose the wrong functional form for the regression model, exclude necessary explanatory variables or include unnecessary ones.  

### 4. Constant variance of $u_i$ (homoscedasticity)  

This assumption states that the variation of $u_i$ is the same regardless of $X$.  

$$
\begin{align}
var(u_i) & = E[u_i - E(u_i|x_i)]^2 \\
& = E(u_i|X_i)^2 \\
& = E(u_i^2) \\
& = \sigma^2
\end{align}
$$  

With this assumption, we assume that the variance of $u_i$ for a given $X_i$ (i.e. conditional variance) is a positive constant value given by $\sigma^2$. This means that the variance for the various $Y$ populations is the same. What this also means is that the variance around the regression line is the same and it neither increases nor decreases with change in values of $X$.  

### 5. No autocorrelation between disturbances  

Given any two $X$ values, $X_i$ and $X_j$ ($i \neq j$), the correlation between any two $u_i$ and $u_j$ is zero. In other words, the observations are independently sampled. Notationally, $cov(u_i, u_j|X_i, X_j) = 0$. This assumption states that there is no **serial correlation** or **autocorrelation** i.e. $u_i$ and $u_j$ are uncorrelated.  

To build an intuition, let's consider the population regression function $Y_t = \beta_1 + \beta_2X_t + u_t$   

Now suppose that $u\_t$ and $u\_{t-1}$ are positively correlated. This means that $Y\_t$ depends not only on $X\_t$ but also on $u_{t-1}$ because that affects $u_t$. Autocorrelation occurs in timeseries data like stock market trends where the the observation of one day depends on the observation of the previous day. What we assume is that each observation is independent as it is in case of the income-expense example we saw.  

### 6. The sample size $n$ must be greater than the number of parameters to estimate  

This is fairly simple to understand. To be able to estimate $\beta_1$ and $\beta_2$, we need atleast 2 pairs of observations.

### 7. Nature of X variables  

There are a few things we assume about the value of $X$ variables. One, for a given sample they are not all the same i.e. $var(X)$ is a positive number. Second, there are no outliers among the values.  

If we have all the values of $X$ the same in a given sample, the regression line would be a horizontal line and therefore it will be impossible to estimate $\beta_2$ and thus $\beta_1$. This will happen because in the equation $\hat{\beta}_2 = \frac{\sum{x_iy_i}}{\sum{x_i^2}}$, the denominator will become zero since $Xi$ and $\bar{X}$ will be the same.

Furthermore, we assume that there are no outliers in the $X$ values. Suppose there are a few $X$ values which are far apart from the mean value, the regression line which we get will be vastly different depending on whether the sample drawn contains these outliers or not.  

These are the 7 assumptions underlying OLS. In the next section, I'd like to elucidate on assumption 3 and 5 a little more.  

## More on Assumption 3. and 5.  

In assumption 3. we state that $E(u_i|X_i) = 0$ i.e. on average the disturbance term is zero. For the expected value of a variable $u_i$ given another variable $X_i$ be zero means the two variables are uncorrelated. We assume these two variables to be uncorrelated so that we can assess the impact each has on the predicted value of $Y_i$. If the disturbance $u_i$ and the explanatory variable $X_i$ were correlated positively or negatively then it means that $u_i$ contains some factor that affects the predicted value of $Y_i$ i.e. we've skipped an important explanatory variable that should've been included in the regression model. This means we have functionally misspecified our regression model and introduced specification bias.  

Functionally misspecifying our regression model will also introduce autocorrelation in our model. Consider the two plots shown below.  

{% asset_img Fit_1.png %}
{% asset_img Fit_2.png %}

In the first plot the line fits the sample points properly. In the second one, it doesn't fit properly. The sample points seem to form a curve of some sort while we try to fit a straight line through it. What we get is runs of negative residuals followed by positive residuals which indicates autocorrelation.  

Autocorrelation can also be introduced not just by functionally misspecifying the regression model but also by the nature of data itself. For example, consider timeseries data representing stock market movement. The value of $X\_t$ depends on $X\_{t-1}$ i.e. the two are correlated.   

When there is autocorrelation, the OLS estimators are no longer the best estimators for predicting the value of $Y_i$.  

## Conclusion  

In this post we looked at the assumptions underlying OLS. In the coming post we'll look at the Gauss-Markov theorem and the BLUE property of OLS estimators.  
<hr>

[1] There are also **stochastic regression models** where the values of X are not fixed.
