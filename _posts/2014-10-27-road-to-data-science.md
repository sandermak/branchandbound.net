---
layout: post
title: "The Road to Data Science" 
category : data 
tags : [analysis, r]
excerpt: "Data is invariably a large part of the applications we deliver. Thinking about data, we often are concerned with ACID, scalability and other operational aspects. Building an (enterprise) app means we want our data to be safe, accessible and acted upon by our business rules. Sure, there is the occasional reporting need, but usually nothing a few well-placed queries won't be able to solve."

---
(this article appeared in an edited form in DZone's [Big Data guide](http://java.dzone.com/articles/introducing-dzones-2014-guide-2))

### Shifting sands
But the world has changed. Gmail sorts our priority inbox for us. Facebook decides what's important in our newsfeed on our behalf. E-commerce sites are full of recommendations, sometimes eerily accurate. We see automatic tagging and classification of natural language resources. Ad-targeting systems predict how likely you are to click on a given ad. The list goes on and on. Applications are getting smarter with their data. It's not just about managing and consuming data anymore. It's all about deriving knowledge and insights from existing data, predicting future data and creating a customised user experience. And not just in fancy reports for management (think 'Business Intelligence'), but as an integral part of the application. Data drives the user experience directly, not after the fact. This is what data science is all about. Granted, the term is incredibly hyped, but there's a lot of substance behind the hype. So we might as well give it a name and try to figure out what it means for us as developers.

All the applications discussed above arose in the context of the large web-giants (Google, Yahoo, Facebook) and lots of startups. Yes, these places are filled to the brim with very smart people, working on the bleeding edge. But make no mistake. This trend will trickle down into 'regular' application development just as well. Users' expectations for business applications increase, if only because they interact with slick and intelligent apps every day at home. For enterprise applications it's not a matter of if, but when.

### From developer to data scientist
How do we cope with these increased expectations? It's not just a software engineering problem. You can't just throw libraries at it and hope for the best. Yes, there are great machine learning libraries, like [Apache Mahout](https://mahout.apache.org/) (Java) and [scikit-learn](http://scikit-learn.org/) (Python). There are even programming languages squarely aimed at doing data science, such as the [R language](http://r-project.org). But it's not just about that. There is a more fundamental level of understanding you need to attain to wield these tools. 

No, this article will not be enough to gain this level of understanding. It can, however, show you the landmarks along the road to data science. This diagram (adapted from Drew Conway's [original](http://drewconway.com/zia/2013/3/26/the-data-science-venn-diagram)) shows the lay of the land:

![Data science venn diagramg](/pics/datascience_venn.png)

As software engineers we can relate to hacking skills. It's our bread and butter. And that's good, because from this solid basis you can branch out into the other fields and become more well-rounded. Let's tackle domain expertise first. It may sound obvious, but if you want to create good models for your data, you need to know what you're talking about. This is not strictly true for all approaches. For example [deep learning](http://en.wikipedia.org/wiki/Deep_learning) and other machine learning techniques might be viewed as an exception. In general though, having more domain-specific knowledge is better. So start looking beyond the user-stories in your backlog and talk to your domain experts about what really makes the clock tick. Beware though. If you 'only' know your domain and can churn out decent code, you're in the danger zone. Meaning you're at risk of re-inventing the wheel, misapplying techniques and many other ways of shooting yourself in the foot.

The elephant in the room here is of course math & statistics. The link between math and the implementation of features such as recommendation or classification is very strong. Even if you're not building a recommender algorithm from scratch (which hopefully you wouldn't), you need to know what goes on under the hood in order to select the right one and to tune it correctly. As the diagram points out, the combination between domain expertise and math & statistics knowledge is traditionally the area of researchers or analysts within companies. But when you combine these skills with software engineering prowess, many new doors will open.

What can you do as developer if you don't want to miss the bus? Before diving head-first into libraries and tools, there are several areas where you can focus your energy:

- Data management
- Statistics
- Math

We'll look at each of them in the remainder of this article. Thinks of these items as the major stops on the road to data science.

### Data management
Features like recommendation, classification and prediction cannot be coded in a vacuum. You need data to drive the process of creating/tuning a good recommender engine for your application, in your specific context. So it all starts with gathering relevant data. It might already be in your databases, or you might have to setup new ways of capturing relevant data. Then comes the act of combining and cleaning data. This is also known as data wrangling or munging. Different algorithms have different pre-conditions on input data. You'll have to develop a good intuition for good data versus messy data.


Typically, this phase of a data science project is very experimental. You'll need tools that help you quickly process lots of heterogeneous data and iterate on different strategies. Real world data is ugly and lacks structure. Dynamic scripting languages are often used for these tasks because they fit this challenge perfectly. A popular choice is Python with [Pandas](http://pandas.pydata.org) or the R language. 

It's important to keep a close eye on everything related to data munging. Just because it's not production code, doesn't mean it's not important. There won't be any compiler errors or test failures when you silently omit or distort data, but it will influence the validity of all subsequent steps. Make sure you keep all your data management scripts, and both mangled and unmangled data. You can then always trace your steps. Garbage in, garbage out applies as always.


### Statistics
Once you have data in the appropriate format, the time has come to do something useful with it. But your data is typically just a sample. You want to create models that handle yet unseen data. How can you infer valid information from this sample? How do you even know your data is representative? Enter the domain of statistics, a  vitally important part of data science. I've heard it put like this: 'a Data Scientist is a person who is better at statistics than any software engineer and better at software engineering than any statistician'.


What should you know? Start by mastering the basics. Understand probabilities and probability distributions. When is a sample large enough to be representative? Know about common assumptions such as independence of probabilities, or when values are expected to follow a normal distribution. Many statistical procedures only make sense in the context of these assumptions. How do you test the significance of your findings? How do you select promising features from your data as input for algorithms? Any introductory material on statistics can teach you this. Then, move on the Bayesian statistics. It will pop up more and more in the context of machine learning.


It's not just theory. Did you notice how we conveniently glossed over the 'science' part of data science up till now? Doing data science is essentially setting up experiments with data. Fortunately, the world of statistics knows a thing or two about experimental setup. You'll learn that you always should divide your data in a training set (to build your model) and test set (to validate your model). Otherwise, your model is no good for unseen data: the problem of  overfitting. Even then, you're still susceptible to pitfalls like [multiple testing](http://en.wikipedia.org/wiki/Multiple_comparisons_problem). There's a lot to take into account.

### Math

Statistics tells you about the when and why, but for the how, math is unavoidable. Many popular algorithms such as linear regression, neural networks and various recommendation algorithms all boil down to math. Linear algebra, to be more precise. So brushing up on vector and matrix manipulations is a must. Again, many libraries abstract over the details for you. But it is essential to know what is going on behind the scenes, in order to know which knobs to tune. When results are different than expected, you need to know how to debug the algorithm. 

It's also very instructive to try and code at least one algorithm from scratch. Take linear regression for example, implemented with gradient descent. You will experience the intimate connection between optimization, derivatives and linear algebra when researching and implementing it. Andrew Ng's [Machine Learning class](https://www.coursera.org/course/ml) on Coursera takes you through this journey in a surprisingly accessible way.

### But wait, there's more...
Besides the fundamentals discussed so far, getting good at data science includes many other skills as well. Such as clearly communicating results of data-driven experiments. Or scaling whatever algorithm or data munging method you selected across a large cluster for large datasets. Also, many algorithms in data science are 'batch-oriented', requiring expensive recalculations. Translation into online versions of these algorithms is often necessary. Fortunately, many (open-source) products and libraries can help with the last two challenges.


Data science is a fascinating combination between real-world software engineering and math & statistics. This explains why the field is currently dominated by PhDs. On the flipside, we live in an age where education has never been more accessible. Be it through MOOCs, websites or books. If you want read a hands-on book to get started, read [Machine Learning for Hackers](http://amzn.to/1kE1HB2). Then, move on to a more rigorous book like [Elements of Statistical Learning](http://amzn.to/1pB2ykA). There are no shortcuts on the road to data science. Broadening your view from software engineering to data science will be hard, but certainly rewarding.