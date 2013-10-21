---
layout: post
title: "A failed experiment: improving the Builder pattern" 
category : java 
tags : [java, developers]
excerpt: "Sometimes, you want to try something new. Like last week, when I wanted to implement a builder for a domain object. In Java, lest you wonder why the code samples in this post are so long."
---

The rationale for wanting builders for domain objects goes roughly like this. You want:

- domain objects that are never in an inconsistent state
- immutable domain objects, preferably
- to avoid ['telescoping' constructors'](http://www.captaindebug.com/2011/05/telescoping-constructor-antipattern.html) taking all combinations of optional fields on your object upfront
- a nice (fluent) API for building your domain objects

Granted, you could just use Scala's [case classes](http://docs.scala-lang.org/tutorials/tour/case-classes.html) with named parameters and call it a day. Alas, this was no such day.

### On the shoulders of giants
Obviously the builder pattern has been belaboured by many who are [greater than I am](http://www.amazon.com/gp/product/0321356683/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0321356683&linkCode=as2&tag=branandboun-20). In fact, the original [Gang-of-Four](http://www.amazon.com/gp/product/0201633612/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0201633612&linkCode=as2&tag=branandboun-20) description dates back to 1995. 

But here we are in 2013. Let's say we want a domain object modeling pizza orders. The usual solution looks somewhat like this, using a static inner class to implement the builder:

{% highlight java %}
public class PizzaOrderOldStyle {

    private int size;
    private boolean pepperoni;
    private boolean chicken;
    private boolean mushroom;
    private boolean peppers;
    private String cheese;
    private String sauce;
    private String orderFor;

    public static Builder pizzaOrder(int size, String sauce, String orderFor) 
    {
        return new PizzaOrderOldStyle.Builder(size, sauce, orderFor);
    } 
    
    public static class Builder {
        
        private int size;
        private boolean pepperoni;
        private boolean chicken;
        private boolean mushroom;
        private boolean peppers;
        String cheese;
        private String sauce;
        private String orderFor;

        public Builder(int size, String sauce, String orderFor) {
            this.size = size;
            this.sauce = sauce;
            this.orderFor = orderFor;
        }

        public Builder withPepperoni() {
            this.pepperoni = true;
            return this;
        }

        public Builder withChicken() {
            this.chicken = true;
            return this;
        }

        public Builder withMushroom() {
            this.mushroom = true;
            return this;
        }

        public Builder withPeppers() {
            this.peppers = true;
            return this;
        }

        public Builder withCheese(String cheese) {
            this.cheese = cheese;
            return this;
        }

        public PizzaOrderOldStyle build() {
            return new PizzaOrderOldStyle(this);
        }
    }

    private PizzaOrderOldStyle(Builder builder) {
        size = builder.size;
        pepperoni = builder.pepperoni;
        chicken = builder.chicken;
        mushroom = builder.mushroom;
        peppers = builder.peppers;
        cheese = builder.cheese;
        sauce = builder.sauce;
        orderFor = builder.orderFor;
    }

// Omitted getters
// Omitted equals/hashCode
}
{% endhighlight %}

It's fairly straightforward. The constructor for ```PizzaOrderOldStyle``` is private and takes an instance of the builder. In this constructor, the fields on the domain object are initialized to the values from the builder. Only ```Builder``` can be instantiated by the user of the API and it takes the non-optional values directly. The ```with*()``` methods on the builder expose a [Fluent API](http://en.wikipedia.org/wiki/Fluent_interface) by returning ```this```. Since the resulting domain object has no setters, it is effectively immutable after it is returned from ```build()```.

Building the domain object is now as simple as:

{% highlight java %}
pizzaOrder(10, "tomato", "Sander")
    .withPepperoni()
    .withMushroom()
    .withCheese("parmesan").build();
{% endhighlight %}
(I added a static convenience method ```pizzaOrder``` to instantiate the builder)

### Problems
While the above solution seems reasonable, it contains some flaws. The most obvious one is that this pattern just shifts mutability to the ```Builder``` class. If you don't share builders across threads (which seems reasonable) we can live with this. What bothers me most is that we have three (!) locations where all the properties of the domain model are enumerated. First, the field definitions are repeated inside the ```Builder``` class. Second, the private constructor copies each of domain model properties. Way too many places to screw up. In fact, I only found out through a unit test that I forgot to copy the ```cheese``` property from the builder into the domain object. 

Granted, there are several [IDE plugins](http://code.google.com/a/eclipselabs.org/p/bob-the-builder/) or other forms of code generators that can automate builder generation. Generating code, however, opens up a whole different can of worms.

Can we do better?

### A feeble attempt
Seeing this problem got me thinking. What if the builder is just a façade on top of the domain object? It could use the fields of the domain object as 'intermediate' storage, without exposing the domain object as a whole before it is ready. Java allows non-static inner classes to mess with private state of the outer object. So that was a good starting point:

{% highlight java %}
public class PizzaOrder {

    private int size;
    private boolean pepperoni;
    private boolean chicken;
    private boolean mushroom;
    private boolean peppers;
    private String cheese;
    private String sauce;
    private String orderFor;
    
    private PizzaOrder() {
        // Prevent direct instantiation
    }
    
    public static Builder pizzaOrder(int size, String sauce, String orderFor) 
    {
        return new PizzaOrder().new Builder(size, sauce, orderFor);
    }
    
    public class Builder {
        private AtomicBoolean build = new AtomicBoolean(false);

        public Builder(int _size, String _sauce, String _orderFor) {
            size = _size;
            sauce = _sauce;
            orderFor = _orderFor;
        }

        public Builder withPepperoni() {
            throwIfBuild();
            pepperoni = true;
            return this;
        }

        public Builder withChicken() {
            throwIfBuild();
            chicken = true;
            return this;
        }

        public Builder withMushroom() {
            throwIfBuild();
            mushroom = true;
            return this;
        }

        public Builder withPeppers() {
            throwIfBuild();
            peppers = true;
            return this;
        }

        public Builder withCheese(String _cheese) {
            throwIfBuild();
            cheese = _cheese;
            return this;
        }
        
        public PizzaOrder build() {
            if (build.compareAndSet(false, true)) {
                // check consistency here...                
                return PizzaOrder.this;
            } else {
                throw new IllegalStateException("Build may only be called once!");
            }
        }
        private void throwIfBuild() {
            if (build.get()) {
                throw new IllegalStateException("Cannot modify builder after calling build()");
            }
        }
    }

// Omitted getters
// Omitted equals/hashCode    
}
{% endhighlight %}

Interestingly, the exposed API is identical since we again offer a static convenience function ```pizzaOrder``` hiding the slightly funky ```new PizzaOrder().new Builder(..)``` that is necessary to instantiate the builder:

{% highlight java %}
pizzaOrder(10, "basil", "Fred")
    .withPepperoni()
    .withMushroom()
    .withCheese("mozzarella").build();
{% endhighlight %}

Even though from a syntactic standpoint the API hasn't changed, from a usage perspective there are differences. After a call to ```build()``` any other call on the builder results in an exception. This must be the case, since the underlying fields are the actual fields of the domain object. We don't want those to change after the builder is done. I used an ```AtomicBoolean``` to 'seal' the builder and the underlying domain object. Compare this with the original approach, where you can build as many times as you want with the same ```Builder``` instance. Whether this is a good or bad thing is debatable. In practice it doesn't make much difference since you tend to use a builder only once anyway.

### Builder failure?
So why do I call this a failed experiment? First of all, I expected this solution to be more concise than the original. It isn't. Check the linecounts in [this gist](https://gist.github.com/sandermak/7074352). Indeed, the fields are not enumerated three times. On the other hand, we have to add bookkeeping to manage the state of the builder façade to each ```with*()``` method. It's easy to forget the call to ```throwIfBuild``` and if you do this threatens the immutability of the domain object.

Second, non-nested inner classes keep an implicit reference to their outer containing objects. This means that the builder object itself may prevent the domain object from being garbage-collected. Retaining a reference to the builder in the original pattern doesn't have this problem, since inner static classes are instantiated without an outer instance and hence don't point to an outer instance.

### Pattern failure?
So the result is not groundbreaking. Still, it's a nice variation on the builder pattern. Here's the [gist](https://gist.github.com/sandermak/7074352) containing this post's code if you want to play around with it. More than anything, it reminds us that design patterns only serve to point out [weaknesses in languages](http://c2.com/cgi/wiki?AreDesignPatternsMissingLanguageFeatures). Creating and maintaining builders for every domain object is just too much hassle. Languages like [Scala](http://stackoverflow.com/questions/14813416/functional-programming-domain-driven-design) and [C#](http://www.dotnetperls.com/named-parameters) are much better equipped in this regard.

One improvement I've been thinking of is to use a nested empty instance of the domain class in the builder to store the data. We can gradually build up the object by modifying its private fields until we return it from ```build()```. In this variation the builder can be made static again. In fact, while writing this I decided to [implement this variation](https://gist.github.com/sandermak/7079734) and it looks like the cleanest approach so far. 

Leave a comment if you see other improvements that I missed!