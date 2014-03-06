---
layout: post
title: "Generating JUnit test reports with Jasmine 2.0" 
category : web 
tags : [web, javascript]
excerpt: "While setting up a new project at work, a seemingly small upgrade from Jasmine 1.3 to Jasmine 2.0 broke a vital feature: the ability to generate JUnit XML reports for JavaScript unit tests. Long story short: I created [jasmine2-junit](https://github.com/sandermak/jasmine2-junit) to bring this feature to Jasmine 2.0 again."

---

Jasmine is famous for its soothing HTML based reports:

![Example screen](/pics/jasmine-html.png)

Running your specs in a browser is great during development. You can easily get an overview of your tests and debug failing tests. However, we also want to run the tests in our CI build and have it report success and failure alongside other automated tests. In that case, a pretty HTML page isn't terribly useful. Fortunately, there's a _de facto_ standard for reporting test results: [JUnit XML](http://help.catchsoftware.com/display/ET/JUnit+Format) files. Despite its Java heritage, the JUnit XML format can be (ab)used for all kinds of test results. Most CI servers support it out of-the-box. And to top it off, there's a [Jasmine reporters](https://github.com/larrymyers/jasmine-reporters) project on GitHub that provides a JUnit XML reporter. So all is well, right?

Until you start using Jasmine 2.0, that is.

### Jasmine internals
Jasmine 2.0 was released December 2013 and was pretty much a total overhaul of the project. This [blogpost](http://pivotallabs.com/jasmine-2-0-add-ons/) from a Jasmine team member grudgingly admits that the impact on 3rd party add-ons is significant. Still, the concept of additional reporters stands in Jasmine 2.0. So it's just a matter of porting the 1.3 version to the 2.0 reporter interface, right?

Well, no. That's only part of the story. What happened is that Jasmine 2.0 moved away from having a monolithical implementation with a global ```jasmine``` object by default. That's good. However, Jasmine is now modularized using a roll-your-own JavaScript module pattern implementation. That's not so good. Originally, you could grab the ```jasmine``` global object and register your custom reporter. All you had to do was add the script tag with your own reporter and registration code after including Jasmine's script (see [this example](https://github.com/larrymyers/jasmine-reporters/blob/master/test/junit_xml_reporter.html)). In the new situation, using Jasmine is already a two-step process. First include the core Jasmine scripts. Then, you have to include a [boot.js](https://github.com/pivotal/jasmine/blob/master/lib/jasmine-core/boot.js) script. This script initializes all the Jasmine modules and exposes the Jasmine API on the appropriate scope. But it also runs this gem:

{% highlight javascript %}
var currentWindowOnload = window.onload;

window.onload = function() {
  if (currentWindowOnload) {
    currentWindowOnload();
  }
  htmlReporter.initialize();
  env.execute();
};
{% endhighlight %}

That's right, it directly starts executing the JavaScript tests. Besides the fact that this horribly breaks with AMD module loading (which we use), there's no window to add additional third-party reporters when using the stock ```boot.js``` script. It seems that the idea is to take ```boot.js``` and customize it to your needs. This type of copy/paste/modify development mystifies me. Why not split up assembling and initializing the Jasmine internals and the actual configuration and running of tests in two parts? That would make Jasmine much easier to use in the face of third-party extensions.

### jasmine2-junit
Sorry if that came about a bit ranty, but these things cost too much time to figure out. I just want to write my tests and be happy... But there's good news: the work I did to port the JUnit XML reporter to Jasmine 2.0 and to create a custom ```boot.js``` is now yours for the taking: [jasmine2-junit](https://github.com/sandermak/jasmine2-junit). Setting up the HTML for running your specs is easy:

{% highlight html %}
<!-- Include Jasmine 2.0's assets -->
<link href="lib/jasmine.css" rel="stylesheet" type="text/css">
<script src="lib/jasmine.js"></script>
<script src="lib/jasmine-html.js"></script>

<!-- The JUnit reporter should go before the boot script -->
<script src="../src/jasmine2-junit.js"></script>
<!-- This boot.js is a modified version of Jasmine's default boot.js! -->
<script src="../src/boot.js"></script>

<!-- Include your spec files here -->>
<script src="spec.js"></script>
{% endhighlight %}

You can find a working example in the [repo](https://github.com/sandermak/jasmine2-junit/tree/master/example). Obviously, running this in a browser will not generate any files because the JavaScript sandbox doesn't offer a file API (you'll see a bunch of errors). Though you can still see the visual report of course. To get the JUnit XML output, run a spec file with the provided [runner script](https://github.com/sandermak/jasmine2-junit/blob/master/src/jasmine2-runner.js) for PhantomJS:

{% highlight bash %}
phantomjs jasmine2-runner.js spec.html
{% endhighlight %}

Running this from a CI build is trivial. I also created a [gulp](http://gulpjs.com) plugin to do so. If there's any interest I might push that to NPM as well.

The Jasmine 2.0 JUnit XML Reporter is a bit less configurable than the [original](https://github.com/larrymyers/jasmine-reporters) it was based on. Let me know if you need anything, and we'll make it work.

Happy JavaScript testing!



