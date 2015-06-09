---
layout: post
title: "On monoliths, microservices and critical thinking" 
category : architecture 
tags : [conferences, developers]
excerpt: "Something very interesting happened last week. At first I wasn't sure whether it was a misguided attempt at humor. Sure enough, Twitter timelines switched en masse from advocating microservices to glorifying monoliths. What a strange world we live in."

---

### Microservices
Microservices burst onto the conference scene, blogosphere and other echo-chambers with blunt force in the past years. Like all movements, the microservices movement offers some good points amongst the propaganda. We are dealing with increasingly complex problems at an increasingly large scale. It's good to pause and reflect on these challenges.

Except, that's not what happens. Instead, microservices frameworks, microservices books and other microservices  paraphernalia convert large swathes of developers into 'early adopters' of the golden age. Vendors create microservices platforms, including some sweet lock-in of course. Suddenly we're counting lines of code like a weightwatcher counts calories. Must. Not. Exceed. &lt;insert arbitrary figure here&gt;. It's unhealthy.

### Monoliths
Then, pendulum swings back. Martin Fowler publishes his [MonolithFirst](http://martinfowler.com/bliki/MonolithFirst.html) post. And the people rejoice. It's ok to officially push back against the cool kids again. Now you're scolded for ever starting from scratch with a service oriented architecture. YAGNI, silly! And if you do need it, just chop up your monolith and season to taste. Easy, right? 

As you might have guessed by now, this post is not about technical arguments either for or against monoliths and microservices. If you're looking for that, read Fowler's post. Then, read [Stefan Tilkov's counterpoint](http://martinfowler.com/articles/dont-start-monolith.html) (graciously hosted on Fowler's site as well). I'm much more interested in what this all means for our profession. What does it mean if public software engineering opinion flips 360 degrees in a matter of weeks? It's too easy to chalk it all up to people needing authority figures. Yes, I know: not everybody was all over microservices. But you have to admit there's something fundamentally unsound going on here.

### The art of problem solving
Software development (and by extension, software architecture) is often characterised as a form of problem solving. There's a problem (requirements, input), and we need to create a solution (software, output). It's not wrong per se, but framing it like this primes your thinking in a dangerous way. If there's a problem, there must be an exact solution, right?

![Solution space](/pics/solutionspace.jpg)

However, we all know that there's no single perfect solution. There's a vast _solution space_ and it is our task to navigate it and arrive at a reasonable location in this space. All hypes like microservices are doing, is prematurely pruning our solution space. Without knowing anything about the problem. That's wrong. We all know it. Still, the solution space is often so enormous, it is tempting to think "this time it's different" when a new voice of (apparent) reason appears. Hopefully, the monolith vs. microservices farce brings us back to reality. Critical thinking cannot be done for you.

### Risk management
Instead of problem solving, a much better characterisation of software architecture is risk management. There are trade-offs in every dimension of the solution space. A software architect must quantify, disqualify and hedge the risks. This is a tough job. There's no substitute for experience here. Books like ['The architecture of open-source applications'](http://amzn.to/1Itaaim), containing case-studies, are much more valuable than books on architecture patterns in that sense. Managing risks means building up your architecture from first principles. Not arriving at some architecture through elimination of fashionable trends.

Architectural patterns are tools for navigating the solution space we must explore. Ultimately, the problem we're solving dictates which direction to go. Not our philosophical preference for certain solutions, unless it is based on similar problems we have solved before. If technical debt is like that irresistible car loan - costly but typically harmless -, then choosing the wrong architecture is more like a sub-prime mortgage. Hey, everybody does it. And when you are left homeless, there's nothing else to blame than your own critical thinking skills.