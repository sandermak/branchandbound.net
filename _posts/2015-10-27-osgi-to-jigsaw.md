---
layout: post
title: "From OSGi to Jigsaw" 
category : java 
tags : [modularity, developers, java]
excerpt: "In my [last post](/blog/java/2015/09/java-module-system-first-look) I introduced the new Java Platform Module system with a small example. As of September, there is an implementation to play with, codename Jigsaw. After some introductory toy examples, I did what any curious coder does: port a familiar (and not entirely trivial) codebase to the new shiny technology. In this case, I took a small dynamic dashboard based on OSGi and implemented it using the proposed Java Platform Module System."
---

If you want an introduction on what the new Java Platform Module System (referred to as JPMS hereafter) entails and how to get started, read ['The Java Module system: a first look'](/blog/java/2015/09/java-module-system-first-look). 
This post assumes you're familiar with the basics of the proposed module system. And if you're the kind of person who just wants to see the code: [here you go](https://github.com/sandermak/carprov-jigsaw).

### The original OSGi application
Before diving into the Jigsaw port, let's have a look what the original application is all about:

![Carprov application](/pics/carprov.png)

As you can see, it mimics a car entertainment system dashboard. 
However, the requirement is that this car entertainment system can be dynamically extended and updated. 
Each of the 'apps' come from a separate OSGi bundle. 
The dashboard only collects applications that are currently provisioned to the OSGi runtime and shows them. 
You can click on the icons to access the underlying 'app' (music player, navigaton, phone), which come from the same bundle that contributes the dashboard icon. 
This example has served as demo for a conference talk by myself and my colleague [Paul Bakker](https://twitter.com/pbakker) called 'Provisioning the IoT'. 
In this talk, we use [Apache ACE](https://ace.apache.org) to dynamically update and provision OSGi bundles to running instances of the car entertainment system on multiple devices. 
It's actually really cool to see your system update in real-time without restarting. 
If you want to see it in action I recommend [watching the talk](https://vimeo.com/126446916). 
The demo starts around the 11 minute mark.

![Carprov architecture](/pics/carprov-arch.png)

Technically, the dynamic dashboard looks up all instances of the ```App``` interface in the OSGi service registry. 
This interface is almost the only piece of code that is publicly shared between bundles. 
In turn, bundles containing an ```App``` implementation register themselves upon bundle start, and unregister when the bundle is stopped. 
This makes full use of the dynamic life-cycle afforded by OSGi. 
The dashboard gets ```App``` instances from the service registry without having to know about the implementation classes. 
Inversion of control in action! 
Each app implementation bundle also provides its own resources such as images. 
You can check out the original application [on GitHub](https://github.com/sandermak/carprov).

### Finding the right modules
The question is, how hard is it to port this modularized OSGi application to the JPMS using the Jigsaw prototype? 
Being as dynamic as OSGi isn't a goal of the JPMS.
So to keep expectations in check, I'm already happy if we can port the module and service structure at startup.
Adding and removing new modules dynamically will have to wait for now.

Our challenge is to translate the OSGi bundles into equivalent Jigsaw modules.
The first step for re-creating this example in the JPMS is to find out what should go into the ```module-info.java``` descriptors. 
These module descriptors contain the dependency information for Java modules. 
It is similar to the OSGi meta-data in the manifest file of OSGi jars.

The most straightforward module definition is the one for the API bundle:

{% highlight java %}
module carprov.dashboard.api {
   exports carprov.dashboard.api;
   requires public javafx.graphics;
}
{% endhighlight %}

You can find the full code for the Jigsaw version of the dashboard [on GitHub](https://github.com/sandermak/carprov-jigsaw) if you want to follow along. It compiles and runs on [build b86](http://openjdk.java.net/projects/jigsaw/ea) of the Jigsaw-enabled JDK.

We declare a module with the name ```carprov.dashboard.api```, exporting a package of the same name.
Meaning the interface and helper class inside this package are visible to all modules that import this module.
Next, we need to declare what this module needs in terms of dependencies.
Since the ```App``` interface uses JavaFX types, these need to be required somehow.
An important goal of the JPMS is to modularize the itself JDK as well.
Therefore we cannot just import types from the JDK without specifying which module they come from.
Note that unlike the exports-clause, the requires-clause takes a module name rather than a package name.

So how do you find the right module to require amongst the ~80 modules that currently comprise the JDK in the Jigsaw prototype?
Fortunately, we can do better than trial and error.
The JDK provides a tool called jdeps, which analyzes the dependencies of an existing Java class.
You provide the class name and an appropriate classpath that contains the class:

    $ jdeps -module -cp carprov.dashboard.api.jar carprov.dashboard.api.App
    carprov.dashboard.api -> java.base
    carprov.dashboard.api -> javafx.graphics
       carprov.dashboard.api (carprov.dashboard.api)
          -> java.lang                                          java.base
          -> javafx.scene                                       javafx.graphics

The last two lines indicate that the App interface imports from the java.lang and javafx.scene packages.
By providing the ```-module``` option, jdeps also outputs the source modules (on the far right).
This way, you can identify the modules providing the packages that the analyzed class depends on.
In this case, the dashboard module should require the java.base module and the javafx.graphics module from the JDK.
That's exactly what we did in the module-info.java descriptor earlier.
Except, the java.base module is always implicitly required for all modules.
You can't live without it.

Another option for finding the right modules is to peruse the [module overview](http://cr.openjdk.java.net/~mr/jigsaw/ea/module-summary.html) page of the early access Jigsaw build.
It gives a comprehensive overview of all JDK modules and their dependencies.
To get a feeling for the new modularized Java platform, it's indispensable.

There's on last twist: what does the `public` in `requires public` mean in the module descriptor?
Let's have a look at the App interface:

{% highlight java %}
import javafx.scene.Node;

public interface App {
   String getAppName();
   int getPreferredPosition();   
   Node getDashboardIcon();
   Node getMainApp();
}
{% endhighlight %}

If only the carprov.dashboard.api package would be exported by the Dashboard API module, what happens if another module imports it and tries to use it?
That consuming module is then forced to also require the module containing javafx.scene.Node (in this case javafx.graphics).
Since Node is used as return type in App, the interface cannot be used without access to this class as well.
You could document this as part of the Dashboard API module, but that's error-prone and generally unsatisfactory.
The `public` keyword in the requires-clauses solves this.
Effectively, it re-exports the public packages from the required module as part of the Dashboard API module.
Now, the app implementation modules can require the Dashboard API module without having to worry about requiring javafx.graphics.
Without the public keyword, compilation fails unless the consuming module itself imports the javafx.graphics module.

This re-exporting mechanism solves the same problem that OSGi 'uses-constraints' solve.
It goes a bit further though.
With the re-exporting mechanism in the JPMS, you can create an 'empty' module that acts as a façade.
The public exports in the module descriptor of this empty module can aggregate several other modules.
As an example, you can use this mechanism to split a module into multiple modules without breaking consumers.
They still require the same module, only now it 'delegates' to other modules.

However, we're getting off track.
Back to porting the dashboard example.
How do the apps actually end up on the dashboard using the JPMS?

### Services with ServiceLoader
So far, we've talked about a single module and its dependencies: the dashboard API.
However, the diagram above shows 5 modules in the sample application.
What about the dashboard implementation module, and the App implementation modules?
We explicitly do not want the dashboard to know about the concrete App implementation classes.
It just needs to gather instances of those implementation classes, without doing the instantiation itself.
Loose coupling, remember?

This means we don't require any App implementation modules in the dashboard's module-info:

{% highlight java %}
module carprov.dashboard.jfx {
   requires carprov.dashboard.api;
   requires javafx.base;
   requires javafx.controls;
   requires javafx.swing;

   uses carprov.dashboard.api.App;
}
{% endhighlight %}

The interesting part is the last line of the module descriptor: ```uses carprov.dashboard.api.App;```.
With this uses-clause, we tell the JPMS that we are interested in instances of App interface.
Subsequently, the dashboard can use the [ServiceLoader API](http://cr.openjdk.java.net/~mr/jigsaw/spec/api/java/util/ServiceLoader.html) to retrieve these instances:

{% highlight java %}
Iterable<App> apps = ServiceLoader.load(App.class);

for(App app: apps) {
  renderDashboardIcon(app);
}
{% endhighlight %}

Instances are created by the module system.
Of course, the big question is: how does the module system locate service providers?

Let's look at an example of a module providing an App service.
The Phone module exposes its App implementation as follows:

{% highlight java %}
module carprov.phone {
   requires carprov.dashboard.api;
   requires javafx.controls;

   provides carprov.dashboard.api.App with carprov.phone.PhoneApp;
}
{% endhighlight %}

The magic happens in the last line.
It indicates that we want to expose an App instance, using the concrete PhoneApp implementation class.
Note that PhoneApp's package is not exported.
Nobody can instantiate it but the JPMS, or another class inside the same module.
There is one requirement for a service class: it must have a default no-arg constructor.
You can even provide services and consume them in the same module. See the [actual source](https://github.com/sandermak/carprov-jigsaw/blob/master/src/carprov.dashboard.jfx/module-info.java) of the Dashboard implementation for an example of both a uses and provides-clause in the same module descriptor.

Now the JPMS knows that the dashboard implementation module wants to see App instances, and the Phone module (and others) provide these instances.
If at any time an additional service implementing the App interface is put on the modulepath, the dashboard will pick it up without modifications to the module descriptor.
It's not as dynamic as the original OSGi application, though.
Only after a restart of the whole application (JVM) are these new modules loaded.

For those who know the OSGi service model, statically describing service dependencies is quite a difference.
OSGi services come and go at run-time.
On the one hand, this is more powerful and dynamic.
On the other hand, the JPMS approach could provide errors in case wiring is not possible at startup time.
By the way, the current prototype does not appear to do so.
Declaring a uses-clause on an interface without any implementations on the modulepath at runtime does not trigger any warnings or errors.
It's still on my list to experiment with the [Layer](http://cr.openjdk.java.net/~mr/jigsaw/spec/api/java/lang/reflect/Layer.html) construct of the JPMS.
Let's see how close it can bring us to loading additional modules on-the-fly.

In short, the ServiceLoader mechanism allows us to hide implementations in a modular world.
It's not quite dependency _injection_ but it is a form of inversion of control.
I'm sure dependency injection models will be built upon this foundation.

### Resources
Modules can encapsulate more than just code.
In this application, we need images as well.
Loading resources using [Class.getResourceAsStream](http://cr.openjdk.java.net/~mr/jigsaw/spec/api/java/lang/Class.html#getResourceAsStream-java.lang.String-) still works, with some caveats.
The class calling this method must be in the same module that contains the resource.
Otherwise, null is returned.

The original OSGi implementation delegated loading resources to a helper class in the Dashboard API bundle.
It did this by passing the [BundleContext](https://osgi.org/javadoc/r4v43/core/org/osgi/framework/BundleContext.html)  of the requesting bundle to this helper class. The BundleContext provides access to the bundle and its meta-data.

{% highlight java %}
public static ImageView getImageByFullname(BundleContext bundleContext, String name) {
  URL entry = bundleContext.getBundle().getEntry(name);
  try {
    Image image = new Image(entry.openStream());
    ImageView view = new ImageView(image);
    view.setPreserveRatio(true);
    return view;
  } catch (IOException e) {
    throw new RuntimeException(e);
  }
}
{% endhighlight %}

I tried to emulate this by passing a Class object from the requesting module to a similar helper class in the JPMS version:

{% highlight java %}
public static ImageView getImageT(Class<?> loadingContext, String name) {
  Image image = new Image(loadingContext.getResourceAsStream(name));
  ImageView view = new ImageView(image);
  view.setPreserveRatio(true);
  return view;
}
{% endhighlight %}

However, the access checks do not seem to care about the Class object which ```getResourceAsStream``` is invoked on, but rather on which class is on top of the call-stack.
That's of course the module that contains the helper class, which cannot read resources from the module that called the helper method.
In that case, getResourceAsStream just returns null.
That lead to some interesting NullPointerExceptions and confused looks on my face.
In the end, I just had my requesting modules call ```getResourceAsStream``` and pass the resulting InputStream to the helper instead:

{% highlight java %}
public static ImageView getImage(InputStream stream) {
     Image image = new Image(stream);
     ImageView view = new ImageView(image);
     view.setPreserveRatio(true);
     return view;
}
{% endhighlight %}

After talking to Mark Reinhold at JavaOne, I learned this behavior is by design.
There is an alternative that looks more like the BundleContext solution: you can also pass a ```java.lang.reflect.Module``` to a helper method like the one above.
This reified module instance effectively allows the recipient to do anything they would like with the calling module.
Including getResourceAsStream on that module.

### List loaded modules

![Carprov modules loaded](/pics/carprov-list.png)

The original dashboard had an app that lists the loaded OSGi bundles comprising the whole application.
Naturally, that needs to be ported as well.
There is a new API for introspecting modules of the JPMS.
Using it is fairly straightforward:

{% highlight java %}
Layer layer = Layer.boot();
for (Module m: layer.modules()) {
  if(m.getName().startsWith("carprov")) {
     String name = m.getName();
     Optional<Version> version = m.getDescriptor().getVersion();
     // Show it in the ui
  }
}
{% endhighlight %}

Modules are organized into Layers.
Since we do not specifically create a module Layer ourselves, the loaded modules are part of the boot-layer.
We retrieve this layer, and ask it for all the loaded modules.
Then, we only process modules that start with "carprov", in order to not show JDK modules in the overview.

### Conclusion
It's going to be interesting to see how the current JPMS prototype will morph into a production-ready module system for Java 9.
One thing is sure: it's a big step forward for the Java platform.

All in all I was pleasantly surprised how far I could come with the Jigsaw prototype.
Yes, it is less dynamic than the OSGi original.
On the other hand, it is also vastly less complex.
OSGi service dynamics are cool, but it makes you handle lots of (concurrency) edge-cases.
Do you really need these dynamics all the time?
Nevertheless, my next challenge will be to bring some of the original dynamics back using the JPMS.
Stay tuned!
