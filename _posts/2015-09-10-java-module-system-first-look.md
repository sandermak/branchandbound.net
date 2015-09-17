---
layout: post
title: "The Java Module system: a first look" 
category : java 
tags : [modularity, developers, java]
excerpt: "A module system for Java has been a long time coming. Late 2014, a new JSR requirements document (JSR-376) was created to this end. The changes are slated for Java 9. However, no working prototype was available. Until yesterday, that is. There now is an OpenJDK early access build that includes Project Jigsaw."
---

Together with my co-worker Paul Bakker I gave a talk on the proposed Java Module system at JavaZone yesterday. We had to base this entirely on the [JSR-376 requirements document](http://openjdk.java.net/projects/jigsaw/spec/reqs/2015-04-01) and other tidbits of information floating around. While proposing this talk at the beginning of the year, we definitely thought a prototype would be available to showcase. However, that didn't quite pan out the way we thought. Instead, the prototype was released just hours after our talk ended (phew). Which means some things we say in the talk are already outdated, but the main ideas still stand. If you're completely new to the Java Module system proposal, I recommend you watch [our talk](https://vimeo.com/138736736) before reading on. It explains the current proposal and puts it in a broader context by comparing it to OSGi.

### Why modules?
So what are modules, and why do we want them? If you want an in-depth discussion, read the ['State of the module system'](http://openjdk.java.net/projects/jigsaw/spec/sotms/) or watch [our talk](https://vimeo.com/138736736). For the uninitiated, here's the Cliff's notes version.

Java has jar files. But really, these are just glorified zip-files containing classes which in turn are inside packages. When you assemble and run an application consisting of different jar files (read: every non-trivial application), you put them on the classpath. And then hope for the best. Because there's no way to tell if you put everything on the classpath that your application needs. Or whether you inadvertently put the same classes (in different jars) on the classpath. Classpath-hell (analogous to DLL-hell) is a real thing. This leads to bad situations rearing their ugly head at runtime. Also, the knowledge that a class was ever in a jar file is lost at runtime. The JRE just sees one big collection of classes. But jars need other jars. It's just not encoded explicitly in any form of meta-data at the moment. Ideally, you would also be able to hide implementation classes inside your jar and only expose your public API. The proposed module system for Java aims to solve these issues:

- modules become first-class citizens that can encapsulate implementation details and expose only what is needed
- modules explicitly describe what they offer, and what they need (dependencies), hence dependencies can be verified and resolved automatically during all phases of development

Having such a module system greatly improves maintainability, reliability and security of large systems. Not in the least of the JDK itself. Given such a system, a module graph can be automatically constructed. This graph contains only the necessary modules to run your application.

### Installing JDK9 early access
If you want to follow along with the example code yourself, you need to install the JDK9 early access build that includes the Jigsaw prototype. On OSX, this means extracting the archive, and moving the extracted directory to ```/Library/Java/JavaVirtualMachines/```. Then, you need to adjust your path and ```JAVA_HOME``` environment variable to point to the JDK9 directory. I'm using the excellent [setjdk](http://www.jayway.com/2014/01/15/how-to-switch-jdk-version-on-mac-os-x-maverick/) bash script to switch between Java installations on the command line. You most certainly don't want to use this early access build as your daily Java installation. You can verify that the installation works by executing ```java -version```. The output should read something like:

    java version "1.9.0-ea"
    Java(TM) SE Runtime Environment (build 1.9.0-ea-jigsaw-nightly-h3337-20150908-b80)
    Java HotSpot(TM) 64-Bit Server VM (build 1.9.0-ea-jigsaw-nightly-h3337-20150908-b80, mixed mode)

As long as it includes the phrase Jigsaw, you're good to go. The resulting code for the example coming up can found at [https://github.com/sandermak/jigsaw-firstlook](https://github.com/sandermak/jigsaw-firstlook).

### A small example
You can still use JDK9 in 'legacy-mode' with just classes, jars and the classpath. But obviously we want to work with modules. So we'll create a project that produces two modules, where module1 uses code from module2.

The first thing to do, is to structure your project so that modules are clearly separated. Then, meta-data needs to be added to modules in the form of a ```module-info.java``` file. Our example is structured as follows:

    src\
      module1\
         module-info.java
         com\test\TestClassModule1.java
      module2\
         module-info.java
         com\moretest\TestClassModule2.java

Effectively, this introduces another layer (module1, module2) on top of the package layering that you already do in Java. In these 'module directories', we find the ```module-info.java``` descriptor at the root. Furthermore, note that the two classes are in distinctly named packages.

Let's look at the code for ```TestClassModule1```:

{% highlight java %}
package com.test;

import com.moretest.TestClassModule2;

public class TestClassModule1 {
   public static void main(String[] args) {
     System.out.println("Hi from " + TestClassModule2.msg());
   }
}
{% endhighlight %}

Looks pretty vanilla, right? Nothing related to modules going on here. There is an import for the ```TestClassModule2```, on which the main method later calls the ```msg()``` method:

{% highlight java %}
package com.moretest;

public class TestClassModule2 {
   public static String msg() {
     return "from module 2!";
   }
}
{% endhighlight %}

For now, we'll leave the ```module-info.java``` files empty.

### Compiling Java modules
Now for the next step: actually compiling our modules and associated source-files. To make this work, a new javac compiler flag is introduced:

    javac -modulesourcepath src -d mods $(find src -name '*.java')

This assumes you run the command from the parent directory of the ```src``` dir. The -modulesourcepath flag switches javac into module-mode, rather than 'legacy' mode. The -d flag indicates the output directory for the compiled modules. These are output by javac in an exploded directory format. If we later want to deliver modules as jars, that's a separate step.

So what happens if we execute the above javac invocation? We get errors! 

    src/module1/module-info.java:1: error: expected 'module'
    src/module2/module-info.java:1: error: expected 'module'

The empty ```module-info.java``` files are wreaking havoc here. Some new keywords are introduced for these files, the most important being ```module```. These new keywords are scoped to the module-info.java definition. You can still use variables called ```module``` in other Java source files.

We update the module descriptors to contain the minimal amount of information necessary:

    module module1 { }

and for module2:

    module module2 { }

Now the modules are explicitly named in their definitions, but do not contain any other meta-data yet. Compiling again leads to new errors:

    src/module1/com/test/TestClassModule1.java:3: error: TestClassModule2 is not visible because package com.moretest is not visible

Encapsulation in action! By default, all classes/types inside a module are hidden to the outside world. That's why javac disallows the usage of ```TestClassModule2```, even though it is a public class. If we were still in a flat classpath world, everything would be fine and dandy. Of course we can fix this, by explicitly exposing ```TestClassModule2``` to the outside world. The following changes are necessary in module2's ```module-info.java```:

    module module2 {
      exports com.moretest;
    }

That's not enough. If you compile with this change, you still get the same error. That's because module2 now exposes the right package (and thereby all it's containing public types), but module1 does not yet express its dependency on module2. We can do that by changing module1's ```module-info.java```, too:

    module module1 {
       requires module2;
    }

Requirements are expressed on other modules by name, whereas exports are defined in terms of packages. Much can be said about this choice, but I won't go into this for a first look. After making this change, we have our first successful compilation of a multi-module build using the Jigsaw prototype. If you look inside the ```/mods``` directory, you see the compiled artifacts neatly separated into two directories.  Congratulations!

### Running modular code
Just compiling is not much fun of course. We also want to see the app running. Fortunately, the JRE and JDK have also been made module-aware in this prototype. The application can be started by defining a modulepath rather than classpath:

    java -mp mods -m module1/com.test.TestClassModule1

We point the modulepath to the ```mods``` dir that javac wrote to. Then, -m is used to indicate the initial module that kickstarts the resolving of the module graph. We also tack on the name of the main class that should be invoked, and there we have it:

    Hi from from module 2!

### Future
This first look gives a taste of what you can do with modules in Java 9. There's lots more to explore here. Like packaging: besides jars, there is a new format coming called jmod. The module system also includes a services layer that can bind service providers and consumers through interfaces. Think of it as inversion of control where the module system fulfills the role of service registry. It's also very interesting to see how the module system was used to modularize the [JDK itself](http://openjdk.java.net/jeps/200). This in turn enables nice things like creating a [run-time image](http://openjdk.java.net/jeps/220) that contains just the JDK and application modules that your app needs, nothing more. Lower footprint, more options for whole-program optimization, etc. It's all very promising.

The next step for me is to try and port a sample OSGi application that uses several modules and services to the Java 9 module system. Stay tuned!