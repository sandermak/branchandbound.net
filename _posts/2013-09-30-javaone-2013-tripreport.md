---
layout: post
title: "JavaOne 2013 trip report"
category : conferences
tags : [conferences, java, modularity]
excerpt: "Last week, San Francisco was sprawling with developers. It was that time of the year again: JavaOne 2013 took place. Thousands of Java developers gathered September 22-26 to learn new things and meet new people."
---

I started visiting JavaOne in 2010, just after Oracle took over the reigns from Sun. In that sense, my past four JavaOnes gave a consistent experience. Not incredibly exciting, but lots of good content when you look hard enough at the enormous [content catalog](https://oracleus.activeevents.com/2013/connect/search.ww?eventRef=javaone#loadSearch-event=null&searchPhrase=&searchType=session&tc=0&sortBy=&p=&i(11180\)=20801). Whether this is a good or bad thing is a matter of taste. Sun over-promised and under-delivered, Oracle has a consistent and oftentimes conservative message. At least they deliver, with schedule slippages of months rather than years.

### Announcements
This can be pretty short: there were no big announcements at the JavaOne keynotes this year. It was all about consolidating the progress made in the past few years. Delivering Java EE 7, JavaFX and in the near future Java 8 is certainly something to be proud of. The demo's in the technical keynote were fun and engaging. However, the punch-line was missing, so to say. 

![The venue](/pics/javaone2013.jpg)

_(Just before the keynote in Moscone)_

There was actually one new piece of information: an update on project Avatar. Project Avatar was originally announced without any details in 2011. Last year, it became clear that Avatar is a JavaScript framework to tie the browser and Java EE backends together more easily. This year, it finally became tangible. It is now [open-sourced](https://avatar.java.net/) at java.net. No further story, that was it. Honestly, it feels a bit like this baby was left at the doorstep of the hospital. Who's going to take care of it, and will it ever grow up to be important?

Finally, while not really an announcement, it was interesting to note that Oracle is shifting the attention for Java to the 'Internet of things.' This term is used often to explain their heavy focus on embedded Java development. The cynic might say it's because this is the only part of Java that is traditionally well-monetized (with Java ME), so it makes sense to secure this cashflow for the future. Regardless of the motivation, I'm happy to see that a unified VM for Java 8 for all platforms is in the cards. 

### Java 8
Java 8, and specifically its Lambda's took the center stage. With the GA date set for March 2014, now is the right time to [familiarize yourself](http://cr.openjdk.java.net/~briangoetz/lambda/lambda-state-final.html) with Lambda's. Especially nice is the fact that the collections library [has been revamped](http://cr.openjdk.java.net/~briangoetz/lambda/lambda-libraries-final.html) to support a more functional style. Even cooler, Streams are added on top of the improved collections library, providing stream fusion:

{% highlight java %}
int sum = widgets.stream()
                      .filter(w -> w.getColor() == RED)
                      .mapToInt(w -> w.getWeight())
                      .sum(); 
{% endhighlight %}
In this example, there is only one traversal of the ```widgets``` collection because the operations are performed on a [stream](http://download.java.net/jdk8/docs/api/java/util/stream/Stream.html). Stream fusion means that the implementation collects all the transformations up until a terminal operation (like ```sum```). The terminal operation then forces the evaluation of the preceding chain of expressions with a single pass over the actual underlying collection.

### Conference sessions
As to be expected with a huge amount of parallel tracks, the quality of the sessions varied. I did attend some excellent sessions. Particularly interesting was the talk from one of the Jetty founders. [Jetty 9.1](http://www.eclipse.org/jetty/documentation/current/) pushes the performance boundaries for Java-based webservers, with [SPDY](http://en.wikipedia.org/wiki/SPDY) and [SPDY-Push](http://webtide.intalio.com/2012/06/spdy-we-push/) support. All this is supported on top of the Servlet spec. You only need to [configure](http://www.eclipse.org/jetty/documentation/current/spdy.html) the right endpoints.

One of the other great talks I saw was by Ben Christensen of Netflix. He showed how RxJava enables [functional reactive programming](https://github.com/Netflix/RxJava/wiki#functional-reactive-programming-frp) on the JVM. RxJava ports the [Reactive Extensions](http://rx.codeplex.com) from the .Net world to the JVM world. Slides are [here](https://speakerdeck.com/benjchristensen/functional-reactive-programming-with-rxjava-javaone-2013).

Another cool talk was by the Typesafe guys on [SLICK](http://slick.typesafe.com) (Scala Language-Integrated Connection Kit.) In short, SLICK maps standard Scala collection methods and for-comprehensions to database access code in a type-safe manner. Kind of what Linq-to-SQL does for .Net, but slightly more ambitious. Definitely something to consider when accessing relational databases from Scala code without relying on a more traditional Java-inspired ORM approach. Slides can be found [here](http://slick.typesafe.com/talks/2013-09-25_JavaOne/2013-09-25_JavaOne-Scaling-Scala-to-the-Database.pdf).

### Talking at JavaOne
Last year I had the opportunity to speak at JavaOne for the first time. Since it was an enjoyable experience, fate decided that I should double the amount of talks this year (hopefully this trend will not continue for too many years.) 

My first talk was 'Data Science with R for Java developers' on Tuesday. It was fun to introduce a completely new language to a group of Java developers. Understanding the data that lives in your application deeply is increasingly important. The main takeaway of the talk is that learning R gives you a powerful tool for doing [exploratory data analysis](/blog/data/2013/03/year-blogging-analyzed-r/). Here are the slides: 
<br>
<iframe src="http://www.slideshare.net/slideshow/embed_code/26510955" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" allowfullscreen>embed</iframe>
<br><br>

On Wednesday, I presented 'Modular JavaScript' with my co-worker [Paul Bakker](https://twitter.com/pbakker/). Modular development is something we value. For large Java codebases, we use modular designs [on top of OSGi](/blog/java/2013/07/java-modularity-story/) to enforce runtime modularity as well. In JavaScript land, codebases are getting bigger and bigger too. Hence, you need a principled approach to modular development there as well. In this talk we introduce the primitives for modularity in JavaScript and compare several module systems for JavaScript.
<br>
<iframe src="http://www.slideshare.net/slideshow/embed_code/26558391" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" allowfullscreen>embed</iframe>
<br><br>
During both talks there was a lot of audience interaction, which is always an encouraging sign. All in all, JavaOne 2013 was a great experience. Hope to see you in 2014!