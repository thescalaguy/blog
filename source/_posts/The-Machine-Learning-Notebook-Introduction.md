---
title: 'The Machine Learning Notebook: Introduction'
tags:
  - machine learning
date: 2018-10-24 21:35:58
mathjax: true
---


A while ago I'd started writing a series on machine learning titled [ML for Newbies](https://web.archive.org/web/20181022160209/https://medium.com/ml-for-newbies) which I never got around to finishing. This series of posts is a redoing of the older series with greater mathematical, and theoretical rigor. Some of the content will be ported over from that series but most of it will be new content.  

The aim of the series is to summarize everything that I know about machine learning in an approachable, intuituve manner, and also be notes for myself.  

## What is Machine Learning?  

ML is all about automated learning. It is about creating algorithms that can "learn" from the input given to them. These algorithms can then do a number of tasks such as filter out spam emails, predict the weather, etc. More formally, it’s about converting experience into expertise. The input to the algorithm become its "experience" and the task that it performs becomes its "expertise".  

Although ML is a sub-field of AI (Artificial Intelligence), they’re different. AI focuses on replicating human intelligence whereas ML focuses on complementing human intelligence by doing tasks that fall beyond human capabilities like looking for patterns in massive data sets.  

## How does a Machine Learn?  

Like I mentioned earlier, a machine learns from the input data given to it. The way the machine learns can also be classified as supervised, unsupervised, active, or passive.  

### Supervised Learning  

Let’s say your task is to build a spam filter. You have a large data set of emails, each of which is marked as either "spam" or "not spam". You’ll feed this data set to your algorithm and it’ll learn from it. Later, you’ll input an email and it’ll tell you if it is spam or not. This type of learning is called supervised learning. It is called "supervised" because the algorithm was first taught what is spam and what isn’t by a data set generated manually. You had to supervise its learning process.  

### Unsupervised Learning  

Let’s stay with the spam filter example. Let’s now suppose that the emails in the data set are not marked as “spam” or “not spam”. It’s the task of the algorithm to figure it out somehow. Such learning is called unsupervised learning.  

### Passive Learning  

In passive learning, the algorithm only observes the data provided to it and learns from it, without influencing it in anyway. Suppose now that the spam filter starts out blank with no notion of what is and what isn’t spam. It’s up to the user to mark the incoming emails so that the algorithm can learn from it. This is passive learning.  

### Active Learning  

In contrast, let’s say that the spam filter now looked at the email first, and if it found something suspicious, asked the user to mark it as spam or not. This is active learning.  

## What can ML do?  

There are a number of ML algorithms, each focusing on a different task. These algorithms can do a number of things like:

### Clustering  

In simple terms, clustering is grouping. It’s an unsupervised form of learning in which the algorithm looks at the data and groups the similar pieces of together based on some similarity measure. In the spam filter examples above, when the algorithm, on its own, figured out which of the emails are spam, it clustered them together.  

### Classification  

Classification is about finding out which group a piece of data belongs to. It’s a form of supervised learning because the algorithm needs to know beforehand what each of the group looks like before it can be asked to classify a piece of data. When the spam filter was given a data set with marked emails, all it had to do was to answer the question "is this email spam?". Basically, it was asked to classify that email into a group.  

### Regression  

Regression is about understanding relationships between variables. If one variable changes, how would it affect the other? For example, if the amount of power needed for a phone to stay on increased, how would it affect battery life?  

### Association Rules  

Finding out association rules in data sets is also about finding out relationships between variables. Suppose you have a large data set from an online store, association rules is about finding out what items you are likely to buy based on what you’ve previously bought. For example, if you buy milk and sugar, you’re likely to buy eggs. Your rule then becomes {milk, sugar} => eggs.  

For now, don’t worry about the theoretical rigor of these. We’ll get to that in subsequent posts. That’s all for now. I hope this post was motivation enough for you to start learning ML. :)