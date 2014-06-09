---
layout: post
title: SF Music Hack Day 2014
tags:
- Hackathon
- Music
---

As a SV noobie, I'd recently been turned on to hackathons. To me the name Hackathon is a bit misleading. It's less of a marathon and more like a long game of [plinko](https://www.youtube.com/watch?v=IXMSbmhcc6A) fueled by gallons of coffee and several "Aha!" moments. But after a couple of them I've started to notice some patterns in successful hacks.

###Build Something You Would Use
While hacking together GiveMePlaylist we had the intent of using the finished product ourselves. Both [Colin](https://github.com/cdurant) and I had recently talked about contextual playlists, so we were excited about building something related to the subject. This gave us some advantages over other hacks from the start as we had a clear goal and some solid use cases.

A clear goal yields maximum time utilization, where every line of code gets you closer to a working hack. Backtracking screws you in two ways
1. The time lost writing code for the old goal
2. The time to context switch and reorientation towards the new goal

It's pretty clear to see how #1 can set development back, but often #2 can be just as harmful. Being in the middle of context switch during a hackathon is pretty damn stressful, and poorly reorienting yourself can lead to more backtracking.

###Expect Something To Go Wrong
A hack is often a mashup of APIs that weren't made for each other, or a ball of components and wires shoved into a box. When you move at the pace needed for most hacks, something is going to break along the way.
> Start writing your code in small functions so they'll be easy to debug later.
This doesn't mean that you should be writing tests or doing TDD, but making it easy to identify problematic code. Debugging large monolithic functions is the devil.

When you're laying out the hack make sure you've got multiple options for APIs and _especially_ API wrappers. If you're using Node or Meteor make sure the packages you plan to use have documentation and the endpoints you want are implemented. The last thing you want to do is dig through the source and find out you need to call the endpoints directly. If you're using a new or uncommon API become familiar with the request and response formats beforehand.

###Know Your Audience
Take some time to look into the sponsors, the judges, and the other participants. Using a sponsor's API or service will not only get their attention, but will most likely give you the added benefit of their support.

This was crucial during the SF Music Hack. A conversation with one of their engineers saved us multiple hours of work. He took the time to understand what we were trying to accomplish and suggested a better way to complete the task. (Thanks [Echonest](http://the.echonest.com/)!)

Looking into the judges will help prioritize the tasks for a hack. Often the judges are subject matter experts and will be sensitive to certain areas. With this key information it will be more clear what areas of your hack need more attention.

Get to know the people behind the hack. These are people who've gone through much of the same struggle and often have wonderful insights. Chances are you'll run into them again at another hackathon or meetup.

###A Little Bit About GiveMePlaylist
GiveMePlaylist was the product of several conversations about what affects song choice in playlists. The idea is to get a feel for the person's influencers (internal and external) and use the information to generate a better playlist. For GiveMePlaylist we picked 2 key influencers, mood and current weather. (At some point I'll go into the details, but for now [here's the github repo](https://github.com/hharnisc/givemeplaylist))

SF Music Hack Day turned out to be a blast. We meet a ton of great people, took home some cool stuff, [and got recognized by the twitter crew](https://blog.twitter.com/2014/at-musichackday-sf-2014).

<img src="/images/twitter-prize.png" height="50%" width="50%">
