---
title: Max Likelihood
tags:
  - machine learning
date: 2019-04-23 07:58:53
---


So far what we've been doing is point estimation using OLS. Another method of estimation with more mathematical rigor is max likelihood estimation. In this post we will look at likelihood, likelihood function, max likelihood, and how all this ties together in estimating the parameters of our bivariate linear regression model.

## Probability  

When we think of probability, we are thinking about the chance of something happening in the future. For example, if a coin is tossed, what is the chance of getting a head? It's $\frac{1}{2}$.   

But how did we arrive at the conclusion that the probability of getting a head is $\frac{1}{2}$? This leads us to two varying schools of thoughts on probability - frequentism and bayesianism.  

If the coin were tossed a 1000 times, about half the times the outcome would be a head and half the times it would be a tail. The more frequently an event happens, the more probable / certain it is. This is the frequentist definition of probability. Under this definition, probability is the ratio of the number of times we make the desired observation to the total number of observations.   

In the Bayesian approach, probability is a measure of our certainty about something. In a coin toss, there's a 50% chance of getting a head and a 50% chance of getting a tail because there's only two possible outcomes.

This subtle difference in the definition of probability leads to vastly different statistical methods.    

In the Bayesian definition, each event that occurs follows an observed probability distribution which is based on some underlying true distribution. Calculating the probabilities involves estimating the parameters of the true distribution based on the observed distribution.

## Likelihood  

Likelihood is the hypothetical probability that an event that has already occurred would yield a specific outcome<sup>[[1]](http://mathworld.wolfram.com/Likelihood.html)</sup>.  

For example, if a coin were tossed 10 times and out of those 10 times a person guessed the outcome correctly 8 times then what's the probability that the person would guess equally correctly the next time a coin is tossed 10 times? This is the likelihood. Unlike probability, likelihood is about past events.  

The likelihood of observing the data is represented by the **likelihood function**. The likelihood function is the probability density function of observing the given data. Suppose that we have $n$ observations - $y_1, y_2, \dots y_n$ - of data which are realizations of a random variable $Y$. Let the function $f(Y, \theta)$ represent the probability density function of the random variable $Y$ with parameters $\theta$. The the likelihood of observing $y_1, y_2, \dots y_n$ is given by the likelihood function as:  

$$
\begin{align} 
\mathcal{L}(y_1, y_2, \dots y_n; \theta) &= f(y_1; \theta) f(y_2; \theta) \dots f(y_n; \theta) \\\\
&= \prod \limits_{i = 1}^n f(y_i; \theta)
\end{align}
$$


## Max Likelihood Estimation 

If we were to maximize the likelihood function, we'd end up with the highest probability of observing the data that we have observed i.e. the max likelihood. The data that we have observed follows some distribution we'd like to estimate. This would require us to estimate the parameters $\theta$ such that they maximize the likelihood function i.e. $\hat{\theta} = \mathop{argmax}\limits_{\theta} \mathcal{L}(y_1, y_2, \dots y_n; \theta)$. The estimator $\hat{\theta}$ is therefore the max likelihood estimator of the parameters $\theta$ such that it maximizes the likelihood function.  

The procedure followed in maximizing the likelihood function to compute $\hat{\theta}$ is called max likelihood estimation (MLE). MLE answers the following question: given that the we've observed data which is drawn from some distribution with parameters $\theta$, to what value should we set these parameters in the sampling distribution so that the probability of observing this data is the maximum?  

The mechanics of finding $\hat{\theta}$ is an optimizaition problem involving a little bit of differential calculus. It involves calculating the partial derivative of the likelihood function with respect to each of the parameters in $\theta$ and setting them to zero. This would yield a system of equations which can then be solved to obtain estimates of the parameters which would maximize the function.  

Calculating the partial derivatives like this becomes unwieldy very quickly since the likelihood function $\mathcal{L}$ involves multiplication of lot of terms. We can instead calculate the maximum of **log-likelihood** since it will convert the product term to a summation term. It is defined as $\mathcal{l(y_1, y_2, \dots y_n; \theta)} = log(\mathcal{L}(y_1, y_2, \dots y_n; \theta))$.  

This provides us with enough background to apply MLE to find our regression parameters.  

## MLE for Regression  

In regression, we have our data $Y_1, Y_2 \dots Y_n$ which we assume is normally distributed with mean $\beta_1 + \beta_2 X_i$ and variance $\sigma^2$. The likelihood function for this can be written as:  

$$
\begin{align}
\mathcal{L}(Y_1, Y_2, \dots Y_n; \beta_1, \beta_2, \sigma^2) &= f(Y_1, Y_2, \dots Y_n; \beta_1 + \beta_2 X_i, \sigma^2) \\\\
&= f(Y_1; \beta_1 + \beta_2 X_i, \sigma^2) f(Y_2; \beta_1 + \beta_2 X_i, \sigma^2) \dots f(Y_n; \beta_1 + \beta_2 X_i, \sigma^2) \\\\
&= \prod \limits_{i = 1}^{n} f(Y_i; \beta_1 + \beta_2 X_i, \sigma^2) \\\\ 
&= \frac{1}{\sigma ^ n (\sqrt{2 \pi}) ^ n} exp{ \lbrace - \frac{1}{2} \sum{ \frac{(Y_i - \beta_1 - \beta_2 X_i) ^ 2} {\sigma ^ 2}  } \rbrace  }
\end{align}
$$

In the equations above, $f(Y_i) = \frac{1} {\sigma \sqrt{2 \pi}} exp\lbrace -\frac{1}{2} \frac{(Y_i - \beta_1 -\beta_2 X_i)^2} {\sigma ^ 2} \rbrace$ because we assume $Y_i$ to be normally distributed. As mentioned previously, computing the log-likelihood is easier. The log-likelihood is given as:  

$$
\begin{align}
\mathcal{l}(Y_1, Y_2, \dots Y_n; \beta_1, \beta_2, \sigma^2) &= log(\mathcal{L}(Y_1, Y_2, \dots Y_n; \beta_1, \beta_2, \sigma^2)) \\\\
&= log \left( \frac{1}{\sigma ^ n (\sqrt{2 \pi}) ^ n} exp{ \lbrace - \frac{1}{2} \sum{ \frac{(Y_i - \beta_1 - \beta_2 X_i) ^ 2} {\sigma ^ 2}  } \rbrace  } \right) \\\\  
&= log \left( \frac{1}{\sigma ^ n (\sqrt{2 \pi}) ^ n} \right) + log \left( exp{ \lbrace - \frac{1}{2} \sum{ \frac{(Y_i - \beta_1 - \beta_2 X_i) ^ 2} {\sigma ^ 2} } \rbrace  } \right) \\\\
&= -n log(\sigma) - \frac{n}{2} log(2 \pi) - \frac{1}{2} \sum{ \frac{(Y_i - \beta_1 - \beta_2 X_i) ^ 2} {\sigma ^ 2} }
\end{align}
$$ 

To find the ML estimators, we need to differentiate the log-likelihood function partially with respect to $\beta_1$, $\beta_2$, and $\sigma ^ 2$ and set the resulting equations to zero. This gives us:  

$$
\begin{align}
\frac{\partial \mathcal{l}}{\partial \beta_1} &= \frac{1}{\sigma ^ 2} \sum{(Y_i - \beta_1 - \beta_2 X_i)}  \\\\
\frac{\partial \mathcal{l}}{\partial \beta_2} &= \frac{1}{\sigma ^ 2} \sum{(Y_i - \beta_1 - \beta_2 X_i)} X_i \\\\  
\frac{\partial \mathcal{l}}{\partial \sigma ^ 2} &= - \frac{n}{2 \sigma ^ 2} + \frac{1}{2 \sigma ^ 4} \sum{(Y_i - \beta_1 - \beta_2 X_i) ^ 2}
\end{align}
$$

Setting the above equations to zero and letting $\tilde{\beta_1}$, $\tilde{\beta_2}$, and $\tilde{\sigma} ^ 2$ represet the ML estimators, we get:  

$$
\begin{align}
\frac{1}{\tilde{\sigma ^ 2}} \sum{(Y_i - \tilde{\beta_1} - \tilde{\beta_2} X_i)} &= 0 \\\\  
\frac{1}{\tilde{\sigma ^ 2}} \sum{(Y_i - \tilde{\beta_1} - \tilde{\beta_2} X_i)}X_i &= 0 \\\\  
\- \frac{n}{2 \tilde{\sigma} ^ 2} + \frac{1}{2 \tilde{\sigma} ^ 4} \sum{(Y_i - \tilde{\beta_1} - \tilde{\beta_2} X_i) ^ 2} &= 0
\end{align}
$$

Simplifying the first two equation above, we get:  

$$
\begin{align}  
\sum{Y_i} &= n\tilde{\beta_1} + \tilde{\beta_2} \sum{X_i} \\\\  
\sum{Y_iX_i} &= \tilde{\beta_1} \sum{X_i} + \tilde{\beta_2} \sum{X_i^2}
\end{align}
$$  

The equations above are the normal equations for OLS estimators. What this means is that the OLS estimators $\hat{\beta_1}$ and $\hat{\beta_2}$ are the same as the ML estimators $\tilde{\beta_1}$ and $\tilde{\beta_2}$ when we assume they're normally distributed. The estimator for homoscedastic variance under ML is given by $\tilde{\sigma} ^ 2 = \frac{1} {n} \sum{\hat{u_i} ^ 2}$. This estimator underestimates the true $\sigma ^ 2$ i.e. it is biased downwards. However, in large samples, $\tilde{\sigma} ^ 2$ converges to the true value. This means that it is asymptotically unbiased.   

## Summary  

In summary, MLE is an alternative to OLS. The method, however, requires that we make an explicit assumption about the probability distribution of the data. Under the normality assumption, the estimators for $\beta_1$ and $\beta_2$ are the same in both MLE and OLS. The ML estimator for $\sigma ^ 2$ is biased downwards but converges to the true value when the sample size increases.
