---
layout: post
title:  "Discover TrackerDog! A generic .NET object change tracker"
date:   2016-01-06 20:44:24 +0100
categories: projects .net
excerpt: "TrackerDog has born as a generic object change tracking framework. It has no bindings with database providers: it's just an in-memory object change tracker"
---

Recently I had to integrate MongoDB into the .NET world. While there's a good official .NET to work with MongoDB it has a drawback compared to working with object-relational mappers (from now, *OR/M*): **popular OR/M frameworks like Entity Framework and NHiberante have object change tracking**.

For example, if you get a persistent object from your favourite database using an OR/M like Entity Framework and you set some property, once you *commit* the *unit of work*, it'll generate an `UPDATE` SQL command for you.

Tracked objects are a good friend when implementing a system with a good separation of concerns in mind: *why your domain would need to know that updating an object isn't setting one or more properties?*. When you implement a system on top of paradigms like *domain-driven design*, you want your *domain objects* to be absolutely agnostic of how they're created, updated, read or deleted in the underlying data store (either if it's a SQL or NoSQL database). That is, your domain creates an *unit of work* and it can detect changes in your objects and once you've committed the whole *unit of work* can generate SQL/NoSQL commands and this is nice because you can concentrate your effort **in developing your product instead of how the data is actually persisted**.

### What happens with MongoDB? 

Well, as I pointed out above, its built-in class-to-document mapping system lacks of change tracking. You need to implement your own approach to execute an insertion or update to your Mongo database. 

**Actually this is also true for other NoSQL databases like Redis and Cassandra (and many others)**. 

Since we're agreed about you prefer to update your objects just setting its properties I believe that as .NET developer you want a solution for this, and **I'd like to share it with the world, because I really think that it can be useful not only in data scenarios but also in many others like data-binding.**

### TrackerDog to the rescue!

<img src="http://mfidemraizer.github.io/trackerdog/media/dogtracker.png">

[TrackerDog][1] has born as a generic object change tracking framework. **It has no bindings with database providers: it's just an in-memory object change tracker.**

For now, it's perfect to implement *unit of work* change tracking and, because tracked objects auto-implement `INotifyPropertyChanged` and collection properties also auto-implement `INotifyCollectionChanged`, [*TrackerDog*][1] can be also used to enable two-way binding on your objects on GUI frameworks like Windows Forms or Windows Presentation Foundation.

Basically, [*TrackerDog*][1] generates proxy objects from your POCOs (Plain Old CLR Objects). For example, given a class `User`:

{% highlight csharp linenos  %}
public class User
{
  public virtual string Name { get; set; }
  public virtual byte Age { get; set; }
}
{% endhighlight %}


...you would be able to track `User` instance changes as follows:

{% highlight csharp linenos %}
// This would be called during application initialization
TrackerDogConfiguration.TrackTheseTypes(Track.ThisType<User>());

// You can turn any already instantiated object into a trackable object
User user = new User().AsTrackable();
// Or you would be able to get trackable instances from scratch using a factory method...
User user2 = Trackable.Of<User>(); 
{% endhighlight %}

...once you've already got a trackable object:

{% highlight csharp linenos %}
// You would produce changes to the whole object:
user.Name = "Matías";
user.Age = 30;
{% endhighlight %}


Now getting changed and unchanged properties is dead simple:

{% highlight csharp linenos %}
IObjectChangeTracker changeTracker = user.GetChangeTracker();

IImmutableSet<IObjectPropertyChangeTracking> changedProperties = changeTracker.ChangedProperties;

// Also you can get unchanged properties
IImmutableSet<IObjectPropertyChangeTracking> unchangedProperties = changeTracker.UnchangedProperties;
{% endhighlight %}


**This is just an intro to [TrackerDog][1]: it implements many other useful and yet powerful features**. [Learn more about TrackerDog on its official API reference site!][1]

[1]: http://mfidemraizer.github.io/trackerdog/