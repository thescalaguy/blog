---
title: Classic Normal Linear Regression Model
tags:
  - machine learning
date: 2019-04-17 11:02:08
---


The posts so far have covered estimators. An estimator looks at the sample data and comes up with an estimate for some true population parameter. However, how do we know how close, say $\hat{\beta}_2$, is to the true $\beta_2$? This is a question of statistical inference. To be able to make an inference, $\beta_2$ would need to follow some distribution. CLRM makes no assumptions about the distribution of data. CNLRM (Classic Normal Linear Regression Model), however, adds the assumption of normality i.e. the data and parameters are normally distributed.  

In this post we will cover the normal distribution, CNLRM, and how the assumption of normality helps us.  

## Normal Distribution  

A probability distribution is a function which provides the probabilities of the outcomes of a random phenomenon. For example, a random variable $D$ could describe the outcomes of rolling a fair dice. The probability distribution would then be a function that would return $\frac{1}{6}$ for each of the outcomes of $D$.

The normal distribution is the most commonly encoutered probability distribution. It states that the averages (or any other statistic like sum or count) calculated by drawing samples of observations from a random variable tend to converge to the normal. The distribution is defined by two parameters - the mean ($\mu$), and the variance ($\sigma^2$). If a random variable $X$ is normally distributed, it is written notationally as $X \sim \mathcal{N}(\mu, \sigma^2)$. The probability density function of normal distribution is given by:  

$$  
f(x ; \mu, \sigma^2) = \frac{1}{\sqrt{2 \pi \sigma^2}} e ^ {- \frac{(x-\mu)^2}{2 \sigma ^ 2}}
$$  

## Distribution of $u_i$, $\hat{\beta}_1$, and $\hat{\beta}_2$

For us to be able to draw any inference about the estimators $\hat{\beta}_1$ and $\hat{\beta}_2$, we will have to make some assumption about the probability distribution of $u_i$. In one of the previous posts we saw that $\hat{\beta}_2 = \frac{\sum{x_iy_i}} {\sum{x_i^2}}$. This can alternatively be written as $\hat{\beta}_2 = \frac{\sum{x_iY_i}} {\sum{x_i^2}} = \sum{k_iY_i}$ where $k = \frac{x_i}{\sum{x_i^2}}$. So we have  

$$
\begin{align}
\hat{\beta}_2 & = \sum{k_i Y_i} \\
& = \sum{k_i (\beta_1 + \beta_2X_i + u_i)} 
\end{align} 
$$ 

The $k_i$, betas, and $X_i$ are fixed and therefore $\hat{\beta}_2$ is a linear function of $u_i$. Under CNLRM, we assume $u_i$ to be to be normally distributed. A property of normally distributed random variables is that their linear functions are also normally distributed. What this means is that since $\hat{\beta}_1$ and $\hat{\beta}_2$ are linear functions of $u_i$, they are also normally distributed.  
As mentioned previously, a normal distribution is described by its mean and variance. This means $u_i$ can also be described by its mean and variance as $u_i \sim \mathcal{N}(0, \sigma^2)$. What this shows us is that we assume, on average, for $u_i$ to be zero and there to be constant variance (homoscedasticity). 

A further assumption we make is that $cov(u_i, u_j) = 0$ where $i \ne j$. This means that the $u_i$ and $u_j$ are uncorrelated and independently distributed because zero correlation between two normally distributed variables implies statstical independence. 

Also note that since $u_i$ is normally distributed, so is $Y_i$. We can state this as $Y_i \sim \mathcal{N}(\beta_1 + \beta_2X_i, \sigma^2) $ 

## Why do we assume normality?  

We assume normality for the following reasons:  

First, the $u_i$ subsume the effect of a large number of factors that affect the dependent variable but are not included in the regression model. By using the central limit theorem we can state that the distribution of the sum of these variables will be normal. 

Second, because we assume $u_i$ to be normal, both $\hat{\beta}_1$ and $\hat{\beta}_2$ are normal since they are linear functions of $u_i$.  

Third, the normal distribution is fairly easy to understand as it involves just two parameters, and is extensively studied.  

## Properties of estimators under CNLRM  

1. They are unbiased.  
2. They have minimum variance.  

Combinining 1. and 2. means that the estimators are efficient - they are minimum-variance unbiased.  

3. They are consistent. This means that as the size of the sample increases, the values of the estimators tend to converge to the true population value.  

4. The estimators have the least variance in the class of all estimators - linear or otherwise. This makes the estimators best unbiased estimators (BUE) as opposed to best linear unbiased estimators (BLUE).

## Misc  

Earlier in the post I mentioned that $\hat{\beta}_2 = \frac{\sum{x_iY_i}} {\sum{x_i^2}}$. This section gives the derivation of this expression. We will look at the numerator for ease of calculation.  

$$
\begin{align} 
\sum{x_i y_i} &=  \sum{(X_i - \bar{X}) (Y_i - \bar{Y})} \\
&= \sum{[X_iY_i - X_i\bar{Y} - \bar{X}Y_i + \bar{X}\bar{Y}]} \\
&= \sum{[Y_i(X_i - \bar{X}) - X_i\bar{Y} + \bar{X}\bar{Y}]} \\ 
&= \sum{Y_ix_i} - \sum{X_i\bar{Y}} + n\bar{X}\bar{Y} \\
&= \sum{Y_ix_i} - n\bar{X}\bar{Y} + n\bar{X}\bar{Y} \\ 
&= \sum{x_i Y_i} 
\end{align}
$$


Finito.
