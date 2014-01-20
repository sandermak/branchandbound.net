---
layout: post
title: "ng-conf 2014: the world's first AngularJS conference"
category : conferences
tags : [conferences, javascript, angularjs]
excerpt: "Last week I attended the world’s first ever dedicated AngularJS conference in Salt Lake City. More than 700 JavaScript developers gathered in the Little America hotel for two days to learn from Angular core-devs and community members."
---

Angular is going mainstream, that’s for sure. It was 'Google IO hard’ to get tickets for ng-conf: they sold out in a few minutes tops. The organizers were overwhelmed by this as well and it wouldn’t surprise me if next year’s ng-conf is significantly bigger. Fortunately, a few days before the conference it was announced that the talks were to be live-streamed. Now they are all available on [YouTube](http://www.youtube.com/user/ngconfvideos) for your viewing pleasure.

### Conference format
I really liked the conference format. It was single-track, so no switching between (and looking for) rooms in between talks. Really amazing what a difference in atmosphere this gives, as everybody has the same experience throughout the day. Also, the talks were short: 20 minutes with 5 minute breaks. You would think that it’s hard to go deep in 20 minutes. And it definitely required a lot of skill from the presenters, but most pulled it off. Another advantage of 20 minute talks is that non-interesting topics or speakers are over before you know it.

![Large ballroom](/pics/ng-conf-hall.jpg)

_(Large ballroom filling up for ng-conf)_

AngularJS seems to attract a very bright and friendly community. Everything went super-smooth even though it was the first time around this conference was organized. Another thing that really worked well was a private Google+ community that the organizers used for communication. It was used heavily throughout the event and proved to be a good way to discuss talks in more depth afterwards.

### Future of Angular
Of course the keynote was delivered by two Angular core-team members. Brad Green and Miško Hevery started by giving some context and history around Angular. For example, I never knew that Angular originally included a whole backend and data-syncing solution as well. Or that the name Angular was chosen because HTML has lots of angle brackets. But however interesting the past is, I am more interested in the future of Angular. Miško gave a very clear overview of what’s in store for Angular 1.3 and beyond. In short, Angular 1.3 will be the Angular for the current browsers and AngularJS 2.0 will be for the future browser:

- embracing [Web Components](http://www.w3.org/TR/components-intro/)
- using ES6 to enhance Angular modules and dependency injection
- using [object.observe](http://wiki.ecmascript.org/doku.php?id=harmony:observe) instead of dirty-checking for two-way databinding

All three points are exciting improvements, the first two for increased modularity and the last for increased performance. Slides for the keynote can be found [here](https://docs.google.com/presentation/d/1rno8HFYcst3nrd6xpruX7r427W5g1RRUL36115OEUnQ/edit#slide=id.g261d5ca55_02) or you can just [watch it](http://www.youtube.com/watch?v=r1A1VR0ibIQ).

![Keynote](/pics/ng-conf-keynote.jpg)

_(Brad and Miško keynoting on the first day)_

As you can guess from the picture above, improved support for mobile is also forthcoming.

### AngularDart
It was interesting to see that none of the attendees reacted favorably to Dart as as JavaScript replacement. Much less the fact that AngularDart was created. The Angular team recognized this sentiment and explained the reasoning behind all this. Internal demand at Google spurred the development of AngularDart. Still, most of the 14 core developers currently working on Angular focus on AngularJS rather than AngularDart. Also, the team sees AngularDart as a fertile breeding ground for new concepts and innovations. In fact, many of the points I described on the future of Angular are prototyped in the AngularDart implementation. All of the successful AngularDart experiments will flow back into AngularJS (2.0). The bottomline is that AngularDart is an important part of Angular's future, even if the current focus is mostly on AngularJS.

### Community
An open source project is only as good as its community is. Angular definitely had a bumpy journey in that regard. For example, the 1.2 release was severely delayed without any communication around it. This turned out to be due to an internal experiment with server-side pre-rendering for Angular, which was ultimately abandoned. The sudden introduction of AngularDart is another example of poor communication of longer term plans. 

A lot of these issues stemmed from the lack of resources within the core-dev team. Pull-requests withered and GitHub issues were sometimes overlooked. In the past few months, a lot has changed. Besides extending the core team to 14 developers, an automated build/verification bot checks all pull-requests. Also, the Angular team has weekly issue triage meetings so nothing gets overlooked. Finally, an Angular working group will be assembled consisting of non-core developers, giving the community a voice to steer the direction of Angular. If you'd like to be an active participant in the Angular community, do check out [angularjs.org/i-want-to-help](http://angularjs.org/i-want-to-help).

### Recommended watching
I’ll round off this post by listing some of my favorite talks:

- [Writing a Massive Angular App at Google](http://www.youtube.com/watch?v=62RvRQuMVyg): Google’s DoubleClick team shows their code-organization patterns
- [End to End Angular Testing with Protractor](http://www.youtube.com/watch?v=aQipuiTcn3U): the upcoming integration-testing framework for Angular
- [Angular performance](http://www.youtube.com/watch?v=zyYpHIOrk_Y): some great insights to improve performance of your app.
- [Dependency Injection](http://www.youtube.com/watch?v=_OGGsf1ZXMs): Vojta Ina explains the current prototype of Angular injection based on ES6 classes and annotations.

It's exciting to see that AngularJS is gaining so much momentum. Going to ng-conf was a great way to get inspired to write solid Angular code and to be prepared for what's coming.
