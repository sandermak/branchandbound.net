---
layout: post
title: "Porting an OSGi app to Jigsaw" 
category : java 
tags : [modularity, developers, java]
excerpt: "In my [last post](/blog/java/2015/09/java-module-system-first-look) I introduced the new Java Module system with a small example. As of September, there is an implementation to play with (codename Jigsaw). After some introductory playing around, I did what any curious coder does: port a well-known (and not entirely trivial) codebase to the new shiny toy. In this case, I took a small dynamic dashboard based on OSGi and implemented it using the proposed Java Module System."
---

If you want an introduction on what the new Java Platform Module System (referred to as JPMS hereafter) entails and how to get started, read ['The Java Module system: a first look'](/blog/java/2015/09/java-module-system-first-look). 
This post assumes you're familiar with the basics of the proposed module system.

### The original OSGi application
Before diving into the Jigsaw implementation, let's have a look what the original application is all about:

![Carprov application](/pics/carprov.png)

As you can see, it mimics a car entertainment system dashboard. 
However, the requirement is that this car entertainment system can be dynamically extended and updated. 
Each of the 'apps' come from a separate OSGi bundle. 
The dashboard only collects applications that are currently provisioned and shows them. 
You can click on the icons to access the underlying 'app' (music player, navigaton, phone), which come from the same bundle that contributes the dashboard icon. 
This example has served as demo for a conference talk called 'Provisioning the IoT'. 
Since it is OSGi based, we used [Apache ACE](https://ace.apache.org) to dynamically update and provision bundles to running instances of the car entertainment system on multiple devices. 
It's actually really cool to see your system update in real-time without restarting. 
If you want to see it action I recommend [watching the talk](https://vimeo.com/126446916). 
The demo starts around the 11 minute mark.

![Carprov architecture](/pics/carprov-arch.png)

Technically, the dynamic dashboard looks up all instances of the ```App``` interface in the OSGi service registry. 
This interface is the almost only piece of code that is publicly shared between bundles. 
In turn, the bundles containing an ```App``` implementation register themselves upon bundle start, and unregister when the bundle is stopped. 
This makes full use of the dynamic lifecycle afforded by OSGi. 
The dashboard gets ```App``` instantiations from the service registry without having to know about the implementation classes. 
Inversion of control in action! 
Each app implementation bundle also provides its own resources such as images. 
You can check out the original application [on GitHub](https://github.com/sandermak/carprov).

### Finding the right modules
The question is, how hard is it to port this modularised OSGi application to the JPMS using the Jigsaw prototype? 
And what are the qualitative/quantitative differences between the two approaches? 
The first step for re-creating this example in the JPMS was to find out what should go into the ```module-info.java``` descriptors. 
These descriptors contain the dependency information for Java modules. 
It is similar to the OSGi metadata in the manifest file of OSGi jars.

The most straightforward module definition is the one for the API bundle:

{% highlight java %}
module carprov.dashboard.api {
   exports carprov.dashboard.api;
   requires public javafx.graphics;
}
{% endhighlight %}

We declare a module with the name ```carprov.dashboard.api```, which exports a package of the same name.
This means that the the interface and helper class inside this package are visible to all modules that import this module.
Next, we need to declare what this module itself needs in terms of dependencies.
Since the ```App``` interface uses JavaFX types, these need to be required somehow.
An important goal of the new module system is to modularise the JDK as well.
Therefore we cannot just import types from the JDK without specifying which module they come from.

So how do you find the right module amongst the ~80 modules that currently form the JDK in the Jigsaw prototype?
Fortunately, we can do better than trial and error.
The JDK provides a tool called jdeps, which analyzes a Java class and tells w

   $ jdeps -module -cp mods/carprov.dashboard.api/ carprov.dashboard.api.App
   carprov.dashboard.api -> java.base
   carprov.dashboard.api -> javafx.graphics
      carprov.dashboard.api (carprov.dashboard.api)
         -> java.lang                                          java.base
         -> javafx.scene                                       javafx.graphics

Another option is to peruse the [module overview]() page of the JDK builds.

### Services with ServiceLoader

### List loaded modules

### Conclusion
- Less dynamic
- Less complex

