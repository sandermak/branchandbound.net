---
layout: post
title: "Automatic-Module-Name: Calling all Java Library Maintainers"
category : java
tags : [java, book, modularity]
excerpt: "With the Java 9 release, developers can use the new module system to create modular applications. However, in order to modularize applications, libraries should be usable as modules as well."
---

Creating modular applications using the Java module system is an enticing prospect.
Modules have module descriptors in the form of `module-info.java`, declaring which packages are exported and what dependencies it has on other modules.
This means we finally have explicit dependencies between modules at the language level, and can strongly encapsulate code within these modules.
The book [Java 9 Modularity](https://javamodularity.com) (O'Reilly) written by Paul Bakker and me explains these mechanisms and their benefits in detail.

That, however, is not what this post is about.
Today, we'll talk about what needs to be done to move the Java library ecosystem toward modules.
In the ideal world where all libraries have module descriptors, all is well.
That's not yet where we are in the Java community.

### What You Need To Do Now
We need Java library maintainers to step up!
Ultimately, it would be best for your library to have a module descriptor so that modular applications can depend on it.
Getting there isn't trivial in all cases.
Fortunately, support for the Java module system can be incrementally added to libraries.
This post explains the first step you can take as library maintainer on your way to becoming a Java module.
This first step boils down to picking a module name, and adding it as `Automatic-Module-Name: <module name>` entry to the library's MANIFEST.MF.
That's it.

With this first step you make your library usable as Java module without moving the library itself to Java 9 or creating a module descriptor for the library, yet.
It doesn't even require re-compilation.
So, do yourself a favor and do it now.
If you're not a library maintainer, encourage libraries you use by opening an issue and pointing to this post.
Then, if you'd like to know why this is a good idea and how it actually works, keep reading.

### Automatic Modules
Traditional applications are packaged into JARs and run from the classpath.
Java 9 still supports this, but also opens the door to more reliable and efficient deployment.
Modular applications on Java 9 and later are packaged into JARs which contain module descriptors (_modular JARs_) and run from the module path.
Code in modular JARs on the module path can't access code in traditional JARs on the classpath.
So what happens when a modular application wants to use a library living on the classpath, which doesn't have a module descriptor yet?
The modular application can't express such a dependency in its module descriptor.

It would be unworkable if the only resort for application developers were to wait for the library maintainer to write a module descriptor, or worse, attempt to patch the JAR themselves with a module descriptor.
To prevent this possibly indefinite waiting game, a feature called _automatic modules_ was introduced.
Moving a traditional JAR (without module descriptor) from the classpath to the module path turns it into an automatic module.
Such an automatic module exports all of its packages and depends on all other resolved modules.
Additionally, automatic modules themselves _can_ still access code on the classpath.

So, instead of waiting for libraries to be modularized, application developers can take matters into their own hands.
Traditional JARs can be used as if they were modules.
For an example of automatic modules in action, look at [Paul's post](http://paulbakker.io/java/java9-vertx/) where he uses Vert.x JARs as automatic modules in an application.

### Automatic Module Name Derivation
Still, the question remains how you can express a dependency on an automatic module from application modules.
Where does its name come from?

There are two possible ways for an automatic module to get its name:

- When an `Automatic-Module-Name` entry is available in the manifest, its value is the name of the automatic module
- Otherwise, a name is derived from the JAR filename (see the [ModuleFinder JavaDoc](https://docs.oracle.com/javase/9/docs/api/java/lang/module/ModuleFinder.html#of-java.nio.file.Path...-) for the derivation algorithm)

That second option probably had you shaking your head.
Filenames are not exactly the hallmark of stability, and your library may be distributed in many ways that could lead to different filenames on the user's end. (Maven's standardized approach to artifact naming alleviates this a bit, but is still far from ideal.)
Moreover, the module name derived from a filename might not be your ideal pick as module name.
That's exactly why you're reading this call to action for adding `Automatic-Module-Name` to libraries.
Pick an explicit module name, put it in the `MANIFEST.MF` and ensure a smooth ride into the modular age for users of your library.
This way, you're not forcing users of your library to depend on an 'accidental' module name.

### Naming Library Modules
Naming is hard.
Picking the right module name for your library is important; module descriptors will refer to your library module by this name.
It's effectively part of your API&mdash;once you pick a name, changing it constitutes a breaking change.

For libraries it's essential to pick a globally unique module name.
A long-standing practice in Java is to use reverse DNS notation for packages (e.g. `com.acme.mylibrary.core`).
We can apply the same to module names.
Name your module after the _root package_ of your module.
This is the longest shared prefix of all packages in the module (for the previous example it might be `com.acme.mylibrary`).
Read Stephen Colebourne's [excellent advice](http://blog.joda.org/2017/04/java-se-9-jpms-module-naming.html) on why this is a good idea.
Ensure your module name is valid, meaning it consists of one or more [java identifiers](https://docs.oracle.com/javase/specs/jls/se7/html/jls-3.html#jls-3.8) separated by a dot.

If you want to see examples of libraries who've already gone through this process, look at the module name discussion for [Google Guava](https://github.com/google/guava/pull/2846) and the [Spring Framework](https://spring.io/blog/2017/05/08/spring-framework-5-0-goes-rc1).

### Practical Tips
Most likely your library is built using Maven or Gradle.
Adding a manifest entry to the resulting JAR is a breeze in both build tools.
For Maven, make sure the jar plugin has the following configuration:

```
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-jar-plugin</artifactId>
  <configuration>
    <archive>
      <manifestEntries>
        <Automatic-Module-Name>com.acme.mylibrary</Automatic-Module-Name>
      </manifestEntries>
    </archive>
  </configuration>
</plugin>
```

With Gradle, you can configure the jar plugin as follows:

```
jar {
    manifest {
        name = "mylibrary"
        instruction "Automatic-Module-Name", "com.acme.mylibrary"
    }
}
```

### Sanity-Check Your Library
Is this really all there is to do as a first step toward modularization?
Ideally, yes.
But if you want your library to be used as automatic module on Java 9 and later, there's a few other potential issues you need to verify:

- Make sure your library doesn't use internal types from the JDK (run `jdeps --jdk-internals mylibrary.jar` to find offending code). JDeps (as bundled with Java 9 and later) will offer publicly supported alternatives for any use of encapsulated JDK APIs. When your library runs from the classpath on Java 9 and later, you can still [get away](http://openjdk.java.net/jeps/261#Relaxed-strong-encapsulation) with this. Not so if your library lives on the module path as automatic module.
- Your library can't have classes in the default (unnamed) package. This is a bad idea regardless, but when your library is used as automatic module, this rule is enforced by the module system.
- Your library can't split packages (two or more JARs defining the types in the same package), nor can it redefine JDK packages (`javax.annotation` is a notorious example, being defined in the JDK's `java.xml.ws.annotation` module but also in external libraries).
- When your library's JAR has a META-INF/services directory to specify service providers, then the specified providers must exist in this JAR (as described in the [ModuleFinder JavaDoc](https://docs.oracle.com/javase/9/docs/api/java/lang/module/ModuleFinder.html#of-java.nio.file.Path...-))

Addressing these concerns is a matter of good hygiene.
If you encounter one of these issues and you can't address those, don't add the `Automatic-Module-Name` entry yet.
For now, your library is better off on the classpath.
It will only raise false expectations if you do add the entry.

### What You Need To Do Next
While your library can now be used as automatic module, it isn't really a great module.
Every package is exported and it doesn't express its dependencies yet.
That's why your next step is to add a `module-info.java` file describing which packages must be exported and which dependencies on other modules the library has.
This might even entail some refactoring, by dividing up your code into API (exported) packages and internal packages.

If your library has no external dependencies, creating and compiling this module descriptor is relatively straightforward (see [this real-world example](http://blog.headius.com/2017/10/migrating-to-java-9-modules-maven-osgi.html)).
However, if you have external dependencies, you'll have to wait until those libraries have added module descriptors (or at least have an `Automatic-Module-Name` themselves).
Writing your module descriptor to depend on filename-derived automatic module names is a sure way to break things for your users.
When these transitive dependencies do start modularizing with a different module name, users of your library will experience breakage.

In [Chapter 10](https://javamodularity.com/#features) of our book, we give in-depth advice on how to migrate a library to a proper module step-by-step.
For example, we explain how you add a module descriptor to your library without having to target Java 9+ when compiling your code.
Features like the new `--release` flag and [Multi-Release JARs](http://openjdk.java.net/jeps/238) are very helpful.
That all goes far beyond the scope of this post.
So, please be a good citizen of the Java community.
Decide upon a module name and add it to your library's manifest.
Then, keep the ball rolling by reading up on the module system and adding a real module descriptor.

_Special thanks to Alex Buckley for commenting on a draft version of this post._
