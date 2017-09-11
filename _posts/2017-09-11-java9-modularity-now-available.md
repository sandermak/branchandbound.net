---
layout: post
title: "Java 9 Modularity (O'Reilly) now available"
category : java
tags : [java, book, modularity]
excerpt: "With the imminent release of Java 9, we're happy to announce the O'Reilly book **Java 9 Modularity** is available as ebook, and soon in print as well. More info about how to get the book can be found at [javamodularity.com](https://javamodularity.com)."
---

About a year ago, the Early Release version of Java 9 Modularity was announced by Paul Bakker and me.
It contained a rough version of the first three chapters.
A lot has happened since.
The book has grown to encompass 14 chapters, highlighting all aspects of the upcoming Java module system.
At [javamodularity.com](https://javamodularity.com) you'll find an overview of the chapters, as well as information on how to get the book.

Since the Early Release, the module system implementation itself has seen quite some changes, especially in the area of migration support.
The book has been updated with all recent developments and shows you not only how to create modular applications with Java 9, but also how to approach a migration to Java 9 from earlier versions.
Java 9 Modularity is the definitive guide to the Java module system.

[![Java 9 Modularity Cover](/pics/java9modularity-3d-cover.png)](https://javamodularity.com)


### Why Modules?

The module system has been a boon to the JDK itself.
Once a gigantic monolithic codebase, it is now neatly modularized into recognizable components:

[![Java 9 Modularity Cover](/pics/java.se.ee.small.png)](https://javamodularity.com)

Modularizing the JDK makes it future proof.
Through strong encapsulation in modules, internal packages can truly stay private, allowing evolution of internal code as was envisioned from the start.
Before strong module boundaries, it was all too easy to 'accidentally' rely on non-public APIs.
Explicit dependencies ensure there's backsliding into a big ball of mud.

The nice thing is, with Java 9 you can use the very same module system used to modularize the JDK to modularize your own applications.
Translate those boxes and arrows on your whiteboard into actual Java modules, with module boundaries and dependencies enforced by the Java compiler and runtime.
In the book we approach both new development with Java 9, as well as migration scenarios.

### But I Won't Be Using Modules (Anytime Soon)

Modules are optional in Java 9.
You can keep running applications on the classpath, or you can choose to create modular applications.
However, applications on the classpath still have to run on top of the modularized JDK.
This brings some new challenges when migrating to Java 9.

Especially when the application uses libraries that poke into JDK internals (and there are many of these), you need to know how to address these situations.
The book covers all scenarios you need to handle when migrating existing codebases to Java 9.
For library authors, it is even more important to support Java 9 as soon as possible.
A special chapter on migration of libraries offers help.

### Book Signing Event at JavaOne
If you're at [JavaOne](https://www.oracle.com/javaone/index.html) this year, be sure to stop by at one of our talks as well:

- Designing for Modularity With Java 9 ([CON2606](https://events.rainfocus.com/catalog/oracle/oow17/catalogjavaone17?search=CON2606&showEnrolled=false))
- Modules or Microservices ([CON1450](https://events.rainfocus.com/catalog/oracle/oow17/catalogjavaone17?search=CON1450&showEnrolled=false))
- Migrating to Java 9 Modules ([CON1455](https://events.rainfocus.com/catalog/oracle/oow17/catalogjavaone17?search=CON1455&showEnrolled=false))

Besides these technical sessions, we will also be doing a book signing session at the O'Reilly booth on Wednesday afternoon, October 5th. It will be the first time we hold the print version of the book in our hands as well, so it's going to be special.
Hope to see many people there!
