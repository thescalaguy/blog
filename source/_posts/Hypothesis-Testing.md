---
title: Hypothesis Testing - Intuition
tags:
  - machine learning
date: 2019-09-10 12:26:36
---


Having discussed frequentism and Bayesianism briefly, it's time to move onto hypothesis testing. I was told by an [avid reader of my blog](https://twitter.com/lisamahapatra) that "it reads like a bad textbook". So I am going to try to make it a little more entertaining. 

## What is hypothesis testing?  

Hypothesis testing is how mathematicians settle their bets. Unlike peasants like ourselves who use a coin flip or fist fights, math nerds like coming up with more sophisticated ways to settle their differences. Let's say two mathematicians - John and Nash - are at a bar. John feels duped by Nash after having settled a previous bet with a coin flip. He thinks the coin used by Nash was not fair. Nash insists otherwise. How might they test this hypothesis?  

For a coin to be fair, the probability $p$ of getting any side is $\frac {1}{2}$. We'll call Nash's stance "the null hypothesis" ($H\_0: p = \frac{1}{2}$) and John's stance "the alternative hypothesis" ($H\_1: p \ne \frac{1}{2}$). How do they decide who's right? 

If after flipping the coin 999 times, $T = |\hat{p}\_n - \frac {1}{2}|$ is large, we'll reject the null hypothesis and say that John has been conned because the coin is likely to favor one side other the other. Otherwise, we'll accept the null hypothesis and say that the coin is indeed fair and John is being salty. 

{% asset_img hate.jpg "I hate this idea already." %}

## But really, what is hypothesis testing?

Hypothesis testing is a way to either accept or reject preconceived notions based on data. For example, if we have a sample of people's heights, can we accept that it is drawn from a normal distribution with average height being 170cm? Let's look at it a little more formally.  

Billy has a random variable $X$ with PDF $f(x;\theta)$. $\theta$ is the true population parameter unknown to him.  

A random variable is a black box that prints a number every so often. The internal mechanics of this black box are a mystery lost in the annals of time. What remains known to this day is that the black box is operated by a PDF - a probability density function. It is the PDF that decides the fate of the numbers that come out. The higher the probability of a number, the more often it'll get printed.   

Master craftsmxn of the PDF knew how to tune it carefully using delicate knobs they called "the parameters of the PDF". The more complicated the PDF, the more parameters it has, the more powerful it is. Who made Billy's random variable? To what value is the parameter set? 

The purpose of Billy's life is to achieve nirvana by finding out the true value of $\theta$ - a magic number which may or may not be 42. After a lot of soul searching, traveling to Denver and back, he obtains a sample of size $n$. He diligently calculates $\hat{\theta}$ - his attempt at uncovering the mystery, an estimate for $\theta$.  

During his travels with Jack Kerouac, he had an epiphany that $\theta$ might be ... some value $\theta^\*$ - another number. He's convinced that he's found the answer. But how might he convince the drunks John and Nash that he's right? He'll need to prove statistically that $\hat{\theta}$ indicates that the sample came from a distribution where $\theta = \theta^\*$.  

Could  an estimate calculated from a sample lead Billy to the light? Let's go back to the example of people and their heights. 

Let $X$ be a random variable representing the height of people such that $X \sim \mathcal{N}(\mu, \sigma^2)$. We're told that the standard deviation $\sigma = 15$. From a sample of size $n = 100$, we get the average height $\bar{X} = 165 \text{cm}$. Could this have come from a population with average height $170 \text{cm}$? Let's write this down as $H\_0$ and $H\_1$. 

$$
H_0: \mu = \mu ^ * = 170 \\\\
H_1: \mu \ne 170
$$ 

Our null hypothesis is that yes, this sample came from a population with average height $170 \text{cm}$ and alternative hypothesis is that no, this did not. To test this, we could use one of two approaches - **confidence intervals** or **significance tests** - and so can Billy.  

{% asset_img "cockadoodledo.jpg" "WTF is going on here. Like, seriously" %}

 
### Confidence intervals  

The first method Billy could use is confidence intervals. A confidence interval is an .. interval .. calculated using the estimate $\hat{\theta}$ in which the true value of $\theta$ may lie. The calculation of the interval involves using probability distributions and is best shown with a concrete example.  

In the population example $X$ is distributed normally with sample mean $\bar{X} = 165$. Since $X$ is normally distributed, we can convert it to standard normal distribution with $\mu = 0$ and $\sigma = 1$. We can construct a 95% confidence interval and check whether it contains the value of $\mu$. The standard normal variable $Z$ is calculated as 

$$
Z = \frac {\sqrt{n} ~ (\bar{X} - \mu)} {\sigma}
$$

From the standard normal distribution table, we get $P(-1.96 ~ \le Z ~ \le 1.96) = 0.95$. Let's rewrite this and come up with a formula for calculating the interval. 

$$
\begin{align}
P(-1.96 ~ \le Z ~ \le 1.96) & = 0.95 \\
P(-1.96 ~ \le \frac {\sqrt{n} ~ (\bar{X} - \mu)} {\sigma} ~ \le 1.96) & = 0.95 \\
P\left(\bar{X} - 1.96 ~ \frac{\sigma} {\sqrt{n}} ~ \le \mu ~ \le \bar{X} + 1.96 ~ \frac{\sigma} {\sqrt{n}} \right) & = 0.95
\end{align}
$$  

When we plug in our values from the population example, we get $162.06 \le \mu \le 167.94$. This interval clearly does not include $\mu = 170$ and so we'll have to reject $H\_0$.  
 
I know a lot has been glossed over. Where did $-1.96$ and $1.96$ come from? Why a $95\%$ confidence interval and why not $99\%$? And what puts the "confidence" in confidence interval? All that and much more in the next post. 
 
{% asset_img getout.jpg "Send help, plis" %}

### Significance tests

The second method is called significance tests. Billy already has $\hat{\theta}$ and his assumption that $\theta = \theta^\*$. In this method, Billy can calculate some test statistic and check the probability of getting that statistic under $f(x; \theta^\*)$. If the probability is very low (say below $5\%$ or $1\%$), he'll have to reject $H\_0$ and book flight tickets to Santa Clara. Let's go back to the population example. 

We know that $Z \sim \mathcal{N}(\mu = 0, \sigma ^ 2 = 1)$. Also, we have enough data to calculate the $Z$ statistic, and access to the only surviving copy of the standard normal distribution. We can check what the probability of getting the statistic is and either accept or reject the hypothesis. Let's do that.  

$$
\begin{align}
Z &= \frac {\sqrt{n} (\bar{X} - \mu)} {\sigma} \\
  &= \frac {\sqrt{100} (165 - 170)} {15} \\
  &= -3.33
\end{align}
$$ 

From the standard normal table, the probability of getting $-3.33$ is $0.00043$. This is very low and therefore $H\_0$ has to be rejected. 

{%asset_img enough.jpg "Enough of this bullshit" %} 

## Conclusion  

Billy can achieve nirvana and the drunks John and Nash can settle their bets in one of two ways - confidence intervals or significance tests. I know a lot of terminology has been skipped for the sake of providing an intuitive explanation of the mechanics of the two methods. This I'll cover in the next few posts.  

What we will build towards is using both these methods in the context of linear regression. Our arduous quest lies in finding $\beta\_1$ and $\beta\_2$ from $\hat{\beta}\_1$ and $\hat{\beta}\_2$. 

Finito.
