---
layout: post
title: "Exploring Angular2" 
category : web 
tags : [javascript, angularjs, web]
excerpt: "AngularJS 1.x gained an unprecedented following in the past few years. We use it heavily in our applications, too. All the more reason to dive into the next iteration of this widely popular framework: Angular2."
---

The Angular team is currently chipping away at their backlog, promising a [beta version](https://github.com/angular/angular/milestones) 'real soon now'™. 
Meanwhile, I've been playing around with their alpha releases.
Yes, that is as painful as [it sounds](https://github.com/angular/angular/blob/master/CHANGELOG.md).
Still, it gave me a solid understanding of where the framework is conceptually heading.
Check out [the code](https://github.com/sandermak/ytlive-angular2), then come back for some much-needed context!

The sample app provides an alternative front-end to YouTube.
You can search for live music by providing an artist name.
Search results can either be stored, or played directly.
The playlist is backed by localstorage.
It's not a large application by any means, but it's not a toy application either.
This is 'YouTube live!' in action:

![YT Live example app](/pics/ytlive.png)

### TypeScript
First thing you'll notice when looking at [the code](https://github.com/sandermak/ytlive-angular2) is that YouTube live! is written in TypeScript.
Why? Well, first of all I think TypeScript is a huge improvement over plain JavaScript.
Watch [my talk on TypeScript](https://www.youtube.com/watch?v=sNot2qxYujU) to see why.
This post assumes some familiarity with ES6 and TypeScript.
Again, watch [the talk](https://www.youtube.com/watch?v=sNot2qxYujU) if you want to brush up on that.
It has lots of live coding, so I promise you won't be bored.

The biggest reason for writing Angular2 apps in TypeScript is that Angular2 itself is written in TypeScript.
You can still write Angular2 apps in plain JavaScript (ES5 or ES6), but you'll miss out on some syntactic niceties.
And miss out on full compile-time type-checking of your clientside app.
Trust me, it's a big deal.

After cloning the repo, you can install and run the app with ```npm install && npm start```.
The layout of the application then looks like this:

    src/
    ├── index.html
    ├── tsconfig.json
    ├── node_modules/
    ├── lib/
    └── ytlive/
        ├── playlist/
        │   ├── PlaylistBackend.ts
        │   ├── PlaylistComponents.ts
        │   ├── playlist.html
        │   └── playlistentry.html
        ├── search/
        │   ├── SearchComponents.ts
        │   ├── YTLiveBackend.ts
        │   ├── search.html
        │   └── searchresult.html
        ├── ytlive.html
        └── ytlive.ts

The code is nicely modularised using ES6 modules, supported by TypeScript.
Angular2 does away with its own module system.
This solves the awkward problems of duplicate module definition when combining AngularJS 1.x  with module loaders like require.js.
Notice that Angular2 is distributed as [npm package](https://www.npmjs.com/package/angular2) so installing (and later upgrading) is a breeze.

### Components
When opening up the source files, it is immediately clear that Angular2 is a completely different framework from AngularJS 1.x.
Conceptually you'll recognize some things, but at a technical level it's a clean slate.
If that scares you a bit: I sympathize wholeheartedly.
However, the component-based approach of Angular2 definitely is a step up.
Starting over was a bold move by the Angular team, and it pays off as evidenced by some preliminary [performance figures](http://info.meteor.com/blog/comparing-performance-of-blaze-react-angular-meteor-and-angular-2-with-meteor).

Reminiscent of React, your whole application is constructed as a tree of components:

![YT Live components](/pics/ytlive-components.png)

Angular2 components replace a whole host of abstractions we know from AngularJS 1.x.
It essentially unifies services, controllers and directives.
A component is an annotated class, that can refer to an associated HTML template:

_PlaylistComponents.ts_:
{% highlight javascript %}
import { Component, View, NgFor } from 'angular2/angular2';
import { LocalStoragePlayList } from './PlaylistBackend';

// PlaylistEntryComponent class definition omitted for brevity.

@Component({
  selector: 'playlist',
  providers: [LocalStoragePlayList]
})
@View({
  templateUrl: "ytlive/playlist/playlist.html",
  directives: [NgFor, PlaylistEntryComponent]
})
export class PlaylistComponent {

  constructor(private playlistService: LocalStoragePlayList) { }

  get entries(): ConcertSummary[] {
    return this.playlistService.getPlaylist();
  }

}
{% endhighlight %}

In particular, the @View annotation contains a reference to this template:

_playlist.html_:
{% highlight html %}
<div class="playlist row">
  <div *ng-for="#entry of entries">
    <playlist-entry [entry]="entry"></playlist-entry>
  </div>
</div>
{% endhighlight %}

Together, the template and component class form a reusable whole.
Through the selector property on the @Component annotation, we control how this component can be instantiated in templates.
In the template above, we similarly use the ```playlist-entry``` element to instantiate nested components for each ```entry``` we have in the ```PlaylistComponent```. 
These entries come from the getter method ```entries()``` on that component.
Using the ```[entry]="entry"``` syntax we pass the current entry in the iteration to the nested component instance's ```entry``` property (which is just a plain class member on the PlaylistEntryComponent class).

Note that we use two custom elements in the template: ```ng-for``` and ```playlist-entry```.
Looking at the PlaylistComponent class, you see these are explicitly listed under ```directives``` in the @View annotation.
No more guessing where the 'magic' elements are coming from!
It's right there. And not just as strings, but properly imported and referenced from the file they are defined.
In this case, ng-for hails from Angular2 itself, and PlaylistEntryComponent is defined earlier in the same file (omitted above).
You might be wondering about the slightly funky syntax with asterisks and brackets.
There's a method to the madness, fortunately.
Read [this post](http://victorsavkin.com/post/119943127151/angular-2-template-syntax) for a more in-depth treatment of Angular2 template syntax. And yes, it is 100% valid [HTML attribute syntax](http://www.w3.org/TR/html-markup/syntax.html#syntax-attributes), in case you were wondering.

One fair warning when working with components: component declaration order within a single source file _does_ matter.
I started out defining PlaylistComponent first, and the PlaylistEntryComponent later in the file.
It seemed so logical, but it broke at runtime.
There's a forward reference to a class that doesn't exist yet in the ```directives``` property of PlaylistComponent.
That makes for some nice error messages and stacktraces in the console, I can tell you.
(for the unlucky googler who is suffering from this problem: 'EXCEPTION: Unexpected directive value 'undefined' on the View of component 'PlaylistComponent' was the error with Angular2.alpha45 and earlier)

Moral of the story: define (or import) your components before referencing them in other components. Or resort to [ugly workarounds](http://blog.thoughtram.io/angular/2015/09/03/forward-references-in-angular-2.html).

### Component interaction
So we have a component tree, components encapsulate data and can render themselves initially.
Next question: how does anything get done?
How do components interact with the user and each other?

With AngularJS 1.x, you're used to 2-way databinding by default.
In Angular2, by default data flows uni-directionally, from the root component to the children.
We already saw an example of passing down data to child components through their attributes, which end up on component class members.
This is a one-way street.
You have two main ways of communicating between arbitrary, non-hierarchical components: events, and shared components.

This example uses shared components.
It is also possible to define custom events and trigger behavior throughout the component tree.
However, not all custom events are propagated correctly yet in the alpha-versions I worked with.
You will not find an example of using custom events in YouTube live!, but you can find more information in [this post](http://schwarty.com/2015/08/14/angular2-eventemitter-and-custom-event-name/).

An example of shared components in action is playing a video in YouTube live. 
It's possible to start a video both from the playlist entries and the search results.
This shared functionality can be achieved by simply creating a VideoPlayer class with the appropriate methods and state:

{% highlight javascript %}
export class VideoPlayer {
  public isPlaying = false;
  public currentVideoUrl: string

  public playConcert(id: string) {
    this.isPlaying = true;
    this.currentVideoUrl = this.concertIdToEmbedUrl(id);
  }

  public stop() {
    this.isPlaying = false;
    this.currentVideoUrl = undefined;
  }

  private concertIdToEmbedUrl(id: string): string {
    return yt_embed + id + '?showinfo=0&autoplay=1';
  }
}
{% endhighlight %}

It's just a plain class, no special Angular annotations necessary.
There is no view attached.
One caveat: if we wanted to inject other components into this class, an @Injectable annotation would have been necessary.
We can inject this VideoPlayer class into existing components through their constructors.
It's a bit like services in AngularJS 1.x.

Take for example the SearchResult component, showing the constructor injection:

{% highlight javascript %}
@Component({
  selector: 'search-result',
  properties: ["concert"],
  providers: [LocalStoragePlayList]
})
@View({
  templateUrl: "ytlive/search/searchresult.html",
  directives: []
})
class SearchResultComponent {
  concert: ytbackend.ConcertSummary

  constructor(private playlistService: LocalStoragePlayList,
     private videoPlayer: ytbackend.VideoPlayer) {}

  addToPlaylist(concert: ytbackend.ConcertSummary) {
    this.playlistService.addConcert(concert);
  }

  playConcert(id: string) {
    this.videoPlayer.playConcert(id);
  }
}
{% endhighlight %}

Two things are injected into the constructor: LocalStoragePlayList (so we can save search results) and VideoPlayer (so we can play search results).
In the @Component annotation, you can see that the injection of LocalStoragePlaylist is setup in the ```providers``` property.
But VideoPlayer is not mentioned there. How come?
When you define a provider, that is also the level where the to-be-injected component is instantiated.
This instance is then available to the component _and all its child components_ for injection.
Therefore, the VideoPlayer provider is setup in the root ```YTLiveComponent```.
This way, the same instance of the VideoPlayer is injected into all components that request it in their constructors.
That's good, because there is only one viewport for the videos.
One video can be played at the time, which makes the VideoPlayer is a shared resource that's used by multiple other components.

Playing a concert is as simple as calling the ```playConcert``` method on the SearchResultComponent from the searchresult template:

_searchresult.html_:
{% highlight html %}
<!-- lots of stuff omitted -->
<button title="Play now" class="play btn btn-success">
    <span (click)="playConcert(concert.id)" class="glyphicon glyphicon-play-circle"></span>
</button>
{% endhighlight %}

It binds the click-event on this span to the ```playConcert``` method on the SearchResultComponent, which in turn calls the shared VideoPlayer component.
Binding to DOM-events like this is the primary means of user-interaction.
Obviously, higher-level components are available for easily integration input components et cetera.

The state of the VideoPlayer instance is watched in another component/template:

_search.html_:
{% highlight html %}
<!-- lots of stuff omitted -->
<div *ng-if="playing" id="concerts" class="row">
  <iframe width="100%" height="100%" [src]="embedUrl" frameborder="0" allowfullscreen></iframe>
</div>
{% endhighlight %}

The ```[src]``` syntax binds the src property of the iframe to the ```embedUrl``` property of the component for this template.
If the embedUrl changes, the src of the iframe is automatically updated (but not the other way around).

### Http service
Angular is more than just a front-end component framework. 
In AngularJS 1.x there was an $http service to do backend calls.
The same applies to Angular2.
Instead of returning (their own flavor) of Promises like in 1.x, the new Http component returns [RX Observables](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md).
Angular2 adopts RxJs as core dependency, you see it popping up in several APIs.
It takes some getting used to, but RxJs is a proven library offering a great way to compose asynchronous data flows.

In YouTube live!, we use an injected Http component to do the YouTube API calls:

{% highlight javascript %}
@Injectable()
export class ConcertService {

  private concerts: ConcertSummary[];

  constructor(private http: Http) { }

  public findConcerts(artist: string, duration = Duration.FULLCONCERT): any {
    var ytDuration: string;
    
    // .. snipped for brevity ..

    var searchString = yt_search + ytDuration + '&q=' + encodeURIComponent('live ' + artist);

    return this.http.get(searchString).map((res: any) => {
      var ytResults: {items: YTSearchResult[] } = res.json();
      var transformedResults = ytResults.items.map(this.toConcertSummary)
      this.concerts = transformedResults;
      return transformedResults;
    });
  }
}
{% endhighlight %}

Again, we see a viewless component, but this time with the @Injectable annotation since we need Angular to inject the Http component in the constructor.
After performing a ```get``` call, the result is transformed using ```map``` on the observable.
This returns another observable, now containing data in a format we can use.
One slight annoyance is that the Http.get returns ```any``` in the current typing definition of Angular2.
It would be nice to use the RxJS type definitions for Observables, so we can get some compile-time sanity back here as well.

The resulting Observable is used in the ```searchConcerts``` method on SearchComponent:

{% highlight javascript %}
export class SearchComponent {
  
  private concerts: ytbackend.ConcertSummary[] = [];

  constructor(private concertService: ytbackend.ConcertService,
      private videoPlayer: ytbackend.VideoPlayer) { }

  searchConcerts(): void {
    this.videoPlayer.stop();
    this.concertService
      .findConcerts(this.searchTerm)
      .subscribe((results: ytbackend.ConcertSummary[]) => this.concerts = results);
  }
}
{% endhighlight %}

Since the ConcertService returns an observable, we cannot assign it directly to a class member of type ```ConcertSummary[]```.
Instead, we subscribe to the observable and assign the result once our subscriber is called when results are available.
The template automatically detects changes to ```concerts``` and shows the new results from the API call.
It would be nice if this manual 'unwrapping' of Observables would not be necessary.

### Wrapping up
This post barely scratches the surface of what features are in Angular2.
There's a whole new approach to [Forms](http://blog.ng-book.com/the-ultimate-guide-to-forms-in-angular-2/), a new [Router](https://angular.github.io/router/) and much more.
You will find the documentation to be inadequate though.
There's also lots of outdated information on the web, especially given the pace of the alpha releases and the amount of breakage between releases.
This article itself will be no exception, probably.

Still, a more stable period is forthcoming with the Angular2 beta nearing.
Now is definitely a good time to start learning the concepts of Angular2, but don't expect it to be a beginner-friendly experience.
There's definitely some rough edges to Angular2, but all in all it looks very promising to me.

Play around with [the code](https://github.com/sandermak/ytlive-angular2) for YouTube live and let me know what you think!
