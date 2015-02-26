---
layout: post
title: Reducing Reactive Bugs In Meteor
tags:
- Meteor
---

Since working with meteor in production at [Respondly](http://hharnisc.github.io/2014/08/15/punk-rock-not-corporate-rock.html), a particularly difficult class of bugs has presented itself. It's related to Meteor's fantastic reactive [Tracker](https://www.meteor.com/tracker) package. As more people contribute to a codebase, knowledge tends to spread out across the team making it more difficult for one person to contribute. Here's an example of the problematic pattern:

{% highlight javascript %}
Tracker.autorun(function() {
  var x = Meteor.session.get('session-var'); // reactive
  updateWithSession(x); // ??? not sure if reactive ???
});
{% endhighlight %}

It's not immediately obvious what is reactive in this example. The function **updateWithSession** could be reading from other reactive variables as well. This can cause the **autorun** to fire at unexpected times, having an impact on performance and even causing [timing bugs](http://en.wikipedia.org/wiki/Heisenbug).

_I'm proposing a simple change to better indicate what should trigger the **autorun**:_

{% highlight javascript %}
Tracker.autorun(function() {
  var x = Meteor.session.get('session-var'); // reactive
  Tracker.nonreactive(function(){
    updateWithSession(x); // definitely not reactive
  });
});
{% endhighlight %}

This creates a clear degree of separation of reactive vs non-reactive, and it's also self documenting code.

Now let's say there's something inside **updateWithSession** that you'd like to react to, simply break that out of the function like this:

{% highlight javascript %}
Tracker.autorun(function() {
  var x = Meteor.session.get('session-var'); // reactive
  var y = Meteor.session.get('session-var-2'); // reactive
  Tracker.nonreactive(function(){
    updateWithSession(x, y); // definitely not reactive
  });
});
{% endhighlight %}

It's a little more code, but it will save you time in the long haul. I've found it much easier to come back after a couple months and make changes. I've also found that colleagues pick up the pattern immediately (thanks to [MDGs](https://www.meteor.com/) great naming). Hopefully this saves you some time to add one more little feature!
