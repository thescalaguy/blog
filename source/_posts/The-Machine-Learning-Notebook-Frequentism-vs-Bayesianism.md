---
title: Frequentism vs Bayesianism
tags:
  - machine learning
date: 2019-05-10 11:23:25
mathjax: true
---


When it comes to statistics, there's two schools of thought - frequentism and Bayesianism. In the coming posts we'll be looking at hypothesis testing and interval estimation and knowing the difference between the two schools is important. In this post I'll go over what frequentism and Bayesianism are and how they differ.   

## What is 'probability'?  

The reason for these two schools of thoughts is the difference in their interpretation of probability. For frequentists, probabilities are about frequency of occurence of events. For Bayesians, probabilities are about degree of certainty of events. This fundamental divide in the definition of probability leads to vastly different methods of statistical analysis.  

## The two schools  

The aim of both the frequentists and Bayesians is the same - to estimate some parameters of a population that are unknown.  

The assumption of the frequentist approach is that the parameters of a population are fixed but unknown constants. Since these are constants, no statements of probability can be made about them. The frequentist procedures work by drawing a large number of random samples, calculating a statistic using each of these samples, and then finding the probability distribution of the statistic. This is called the sampling distribution. Statements of probability can be made about the statistic. 

The assumption of the Bayesian approach is that the parameters of a population are random variables. This allows making probability statements about them. There is a notion of some true value that the parameters can take with certain probability. The Bayesian approach thus allows adding in some prior information. The cornerstone of Bayesian approach is Bayes' theorem:  

$$
P(\mathcal{H} ~ | ~ \mathcal{D}) = \frac{P(\mathcal{D} ~ | ~ \mathcal{H}) ~ P(\mathcal{H})} {P(\mathcal{D})}
$$  

Here, $\mathcal{H}$ is the hypothesis and $\mathcal{D}$ is the data. What the theorem lets us calculate is the posterior $P(\mathcal{H} ~ | ~ \mathcal{D})$ which is the probability of the hypothesis being true given the data. The prior $P(\mathcal{H})$ is the probability that $H$ is true before the data is considered. The prior lets us encode our beliefs about the parameters of the population into the equation. $P(\mathcal{D} ~ | ~ \mathcal{H})$ is the likelihood and is the evidence about hypothesis $\mathcal{H}$ given data $\mathcal{D}$. Finally, $P(\mathcal{D})$ is the probability of getting the data regardless of the hypothesis. 

What's important here is the prior $P(\mathcal{H})$. This is our degree of certainty about the hypothesis $\mathcal{H}$ being true. This probability can itself be calculated using frequentist methods but what matters is the fact that Bayesian approach lets us factor it in.   

| Frequentist | Bayesian |
|--|---|
| Parameters are fixed, unknown constants. No statements of probability can be made about them. | Parameters are random variables. Since random variables have an underlying probability distribution, statements of probability can be made about them. |
| Probability is about long run frequencies. | Probability is about specifying the degree of (un)certainty. |
| No statements of probability are made about the data or the hypothesis. | Statements of probability are made about both data and hypothesis. |
| Makes use only of the likelihood. | Makes use of both the prior and the likelihood. |

## The procedures  

In the frequentist approach, the parameters are an unknown constant and it is the data that changes (by repeated sampling). In the Bayesian approach, the parameters are a random variable and it is the data that stays constant (the data that has been observed).  In this section we will contrast the frequentist confidence interval with Bayesian credible interval.  

Both confidence intervals and credible intervals are interval estimators. Interval estimators provide a range of values that the true parameters can take.  

### The frequentist confidence interval  

Let's assume that there's a true parameter $\theta$, representing the mean of a population, that we are trying to estimate. From a sample of values $(x_1, x_2, \dots, x_n)$ we can then construct two estimators $\hat{\theta}_1$ and $\hat{\theta}_2$ and say that the true value $\theta$ lies between $\hat{\theta}_1$ and $\hat{\theta}_2$ with a certain level of confidence.   

To be confident that $\hat{\theta}_1 \le \theta \le \hat{\theta}_2$, we need to know the sampling distribution of the estimator. If the random variable $X$ is normally distributed, then the sample mean $\bar{X}$ is also normally distributed with mean $\mu$ (true mean) and variance $\frac{\sigma^2}{n}$ i.e. $\bar{X} \sim \mathcal{N}(\mu, \frac{\sigma^2}{n})$. In a normal distribution, 95% of the area under the curve is covered by two standard deviations. Therefore, if we want 95% of the intervals to contain the true value of $\theta$, we can construct an interval $\bar{X} \pm 2 \frac{\sigma}{\sqrt{n}}$.  

What this means is that if we were to keep drawing samples and constructing these intervals, 95% of these random intervals will contain the true value of the parameter $\theta$. This can be stated more generally as $P(\hat{\theta}_1 \le \theta \le \hat{\theta}_2) = 1 - \alpha$. This means that with probability $1 - \alpha$ (with $0 < \alpha < 1$), the interval between $\hat{\theta}_1$ and $\hat{\theta}_2$ contains the true value of the parameter $\theta$. So, if we set $\alpha = 0.05$, we're constructing a 95% confidence interval. $\alpha$ is called the level of significance and is the probability of committing type I error.  

Let's suppose we're trying to find the average height of men. It is normally distributed with mean $\mu$ and standard deviation $\sigma = 2.5$ inches. We take a sample of $n = 100$ and find that the sample mean $\bar{X} = 67$ inches. The 95% confidence interval would be $\bar{X} \pm 2 \left( \frac{2.5}{\sqrt{100}} \right)$. This gives us the confidence interval $66.5 \le \mu \le 67.5$. This is one such interval. If we were to construct 100 such intervals, 95 of them would contain the true parameter $\mu$. 

The caveat here is that for simplicity I've assumed the critical value to be 2 instead of 1.96 for constructing the interval.

### The Bayesian credible interval  

A Bayesian credible interval is an interval that has a high posterior probability, $1 - \alpha$, of containing the true value of the parameter. Compared to the frequentist confidence intervals which say that $(1 - \alpha)\%$ of all the intervals calculated will contain the true parameter $\theta$, the Bayesian credible interval says that the calculated interval has $(1 - \alpha)\%$ probability of containing the true parameter. 

Let's suppose we're interested in the proportion of population <sup>[[1]](http://www2.stat.duke.edu/~rcs46/lecturesModernBayes/601-module3-morebayes/lecture5-more-bayes.pdf)</sup> that gets 8 hours of sleep every night. The parameter $\theta$ here now represents proportion. A sample of 27 is drawn of which 11 get 8 hours of sleep every night. Therefore, the random variable $X \sim \text{Binom}(27, ~ \theta)$.   

To calculate a Bayesian credible interval, we need to assume a subjective prior. Suppose the prior was $\text{Beta}(3.3, 7.2)$. The posterior would thus be $\text{Beta}(11 + 3.3, 27 - 11 + 7.2) = \text{Beta}(14.3, 23.2)$. This can be calculated using `scipy` as follows:  

{% codeblock lang:python %}
from scipy import stats
stats.beta.interval(0.9, 14.3, 23.2)
(0.256110060437748, 0.5138511051170076)
{% endcodeblock %}

The 90% credible interval is (0.256, 0.514).  

In summary, here are some of the frequentist approaches and their Bayesian counterparts.  

| Frequentist | Bayesian |
|---|---|
| Max likelihood estimation (MLE) | Max a posteriori (MAP) | 
| Confidence interval | Credible interval |  
| Significance test | Bayes factor | 

A thing that I have glossed over is handling of nuisance parameters. Bayesian procedures provide a general way of dealing with nuisance parameters. These are the parameters we do not want to make inference about, and we do not want them to interfere with the inference we are making about the main parameter. For example, if we're trying to infer the mean of a normal distribution, the variance is a nuisance parameter.

## The critique  

The prior is subjective and can change from person to person. This is the frequentist critique of the Bayesian approach; it introduces subjectivity into the equation. The frequentist approach make use of only the likelihood. On the other hand, the Bayesian criticism of the frequentist approach is that it uses an implicit prior. Bayes theorem can be restated as $\text{posterior} \propto \text{likelihood} \times \text{prior}$. For the frequentist approach, which makes use of likelihood, the prior would need to be set to 1 i.e. a flat prior. The Bayesian approach makes the use of prior explicit even if it is subjective.  

The Bayesian criticism of frequentist procedures is that they do not answer the question that was asked but rather skirt around it. Suppose the question posed was "in what range will the true values of the parameter lie?". The Bayesian credible interval will give one, albeit subjective, interval. The frequentist confidence interval, however, will give many different intervals. In that sense, frequentism isn't answering the question posed.  

Another Bayesian criticism of the frequentist procedures that they rely on the possible samples that could occur but did not instead of relying on the one sample that did occur. Bayesian procedures treat this sample as the fixed data and vary the parameters around it.  

The end.
