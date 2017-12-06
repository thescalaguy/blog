---
title: One Year as a Software Engineer - a Retrospective
tags:
  - misc
date: 2017-12-06 23:04:59
---

December 2017 marks my completing one year as a software engineer. This post is a restrospective where I list down the lessons I've learned, in no specific order of priority. 

## Personal Life

### Learn to invest

> We're having too good a time today. We ain't thinking about tomorrow.
> — John Dillinger, _Public Enemy_  

It's easy to get carried away because of the fat pay cheque your fancy developer job brings you at the end of the month. Think of all the bling you can buy. This hedonistic attitude, however, does not help you hedge against the volatility of the job market. In his book _Out of our Minds: Learning to be Creative_, Ken Robinson states that secure life-long employment in a single job is a thing of the past. This applies even more if you work in a startup environment where things change in the blink of an eye.  

Making proper investments and establishing a second flow of cash can help you get a grip on the situation when it gets rough. The simplest way to invest is to put your money in the stock market and let the dividends add to your monthly flow of cash. You do not have to be an active day trader managing your positions. You can very easily invest in mutual funds, index funds, etc. and they are not that difficult to begin with.   

One of the best books I've been recommended by a friend of mine is [_Trading and Exchanges: Market Microstructure for Practitioners_](https://www.amazon.in/Trading-Exchanges-Microstructure-Practitioners-Association-ebook/dp/B003ZSHIPE?tag=googinhydr18418-21) by Larry Harris. This will give you a very good overview of the market and all that you need to become a confident investor.  

### Be ready to interview  

> Love your job but don't love your company, because you may not know when your company stops loving you.
> — A. P. J. Abdul Kalam, _11<sup>th</sup> President of India_

The key takeaway of working in a startup environment is this: things change very rapidly and when the push comes to shove, you will be thrown overboard. Having seen this happen to people close to me, I've learned that you need to be ready to interview with other startups and/or companies as soon as you are fired or have resigned. This includes staying in touch with the fundamentals you've learned in your CS class like data structures and algorithms, and also knowing the technologies you've used in reasonable depth. Also, having a good network of developers goes a long way in easing your search for a new job. And don't forget to [sharpen the saw](https://blog.codinghorror.com/sharpening-the-saw/) — do things that are not programming that make you a better programmer.  

### Learn to negotiate

> Dude, it’s five minutes.  Let’s un-suck your negotiation.
> — Patrick McKenzie  

Learning how to negotiate is one of the most important skills that is often overlooked. It is overlooked because it is perceived to be cheap to negotiate for salary, or just plain avoiding difficult conversation. Whatever the case, get over it and learn the skill. As Patrick McKenzie mentions in his brilliant blog post [Salary Negotiation: Make More Money, Be More Valued](http://www.kalzumeus.com/2012/01/23/salary-negotiation/), all it takes is five minutes to finalize your salary. These five minutes have a lasting impact for alteast an year to come. 

### Read a lot  

> Read, read, read.
> — William Faulkner

I'll admit that it is hard to take time out to read daily but keeping a dedicated slot of 30 minutes just for reading goes a long way. I make sure it's a distraction-free slot with no dinging notifications and I try not to multitask. Another exercise that I do in conjunction with reading is trying to improve my retention of the material I've read by improving my recall memory. The best way to do this is to use [Feynman technique](https://medium.com/taking-note/learning-from-the-feynman-technique-5373014ad230) where you eludicate what you've learned and pretend to teach it to a student.  

I prefer keeping one programming and one non-programming book as a part of my daily reading. In addition, there's a list of blogs, and papers that I read from time to time. I'll probably post a list as a separate post.  

## Engineering

### Understand Peter principle

> The key to management is to get rid of the managers. 
> — Ricardo Semler  

Laurence Peter came up with the concept that a person in a role keeps getting promoted based on their performance in their current role and not on the abilities that the role demands. It is quite possible to have a manager who doesn't know how to do his job well i.e. he's risen to his level of incompetence. This principle is important to understand as an engineer and is something that should make one reflect on one's current skillset - do you possess the abilities to be in the role you are currently in or do you need to skill up? Again, this goes back to my point on sharpening the saw and doing non-programming things that make you a better developer.  

### Tech debt is evil  

> Simplicity is hard work. But, there's a huge payoff. The person who has a genuinely simpler system - a system made out of genuinely simple parts, is going to be able to affect the greatest change with the least work. He's going to kick your ass. He's gonna spend more time simplifying things up front and in the long haul he's gonna wipe the plate with you because he'll have that ability to change things when you're struggling to push elephants around.
> — Rich Hickey

When Ward Cunningham came up with the tech debt metaphor, he was referring to writing code despite having a poor understanding of the requirements. As time passes, whatever little understanding there was of the requirement fades away and the code is taken for granted. The definition of tech debt has since then come to represent poorly written code that later nobody understands and it's taken for granted - something Ward Cunningham disagrees with.  

The lethal combination is poorly written code for badly understood requirement(s) and you'll come across this very often in startup environments. This comes with some pretty nasty ramifications like team in-fighting, and politics. In worst cases, it can bring the development of new features to a grinding halt.   

Avoiding tech debt has to be a top priority for any startup that wants to grow. A lot of it revolves around establishing some processes to convey requirements among teams and ensuring that the resulting system design is simple. Like the quote by Rich Hickey shows, it is hard work but it will pay off in the longer run. 

### Centralize your Logs  

> However, logging doesn't seem to receive the same level of attention; consequently, developers find it hard to know the 'what, when, and how' of logging.
> — Colin Eberhardt

Please stop asking developers to SSH into machines to read the logs. Centralize your logs by using ELK or if you want to avoid the hassle of setting up ELK, use a third party service like Fluentd or similar. A good centralized logging strategy will not only save you the pain of SSH-ing into multiple servers and `grep`-ing, it will also let you search through them easily. In addition, aggregating logs from various servers helps you identify patterns that may emerge by checking what's happening on multiple servers in a specific time range.  