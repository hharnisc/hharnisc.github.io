---
layout: post
title: "We Wrote A Book: Atomic Migration Strategy For Web Teams"
tags:
- Atomic Design
- Writing
---

It all started at [Fluent Conf 2017](https://conferences.oreilly.com/fluent/fl-ca-2017) with a talk I gave on [Migrating With Atomic Design](https://bufferapp.github.io/buffer-talks/2017/06/09/migrating-with-atomic-design.html#1). After the talk I was approached by O'Reilly and asked if I wanted to turn the talk into a book. My initial reaction was **hell yes**, followed quickly by _what did I just agree to_ 😱. I'm an audio learner and a slow writer, so writing does not come natural to me. However, I really wanted to write a book since I'd never done it before and it seemed like a challenge that would push me outside of my comfort zone. I'd love to share how I went about writing and what went into writing a book with a publisher.

## Getting Help

When thinking about what we had to do at [Buffer](https://buffer.com) to get the migration process going, the more I realized this book needed more than the engineering perspective. The migration would not have happened without [Katie Womersley's](https://twitter.com/katie_womers) tirelessly advocating that a migration was the _right thing to do right now_. Atomic Migration Strategy for Web Teams needed her perspective, to tell the whole story and be useful. So I asked if she wanted to co-author the book (she said yes 🙌). So we set out and created the outline and submitted the proposal to O'Reilly.

## Actually Writing

The writing process, not including editing, took about 5 months to complete between October 2017 and February 2018.

Writing is very draining for me, so my approach to writing is very much about energy management. I had to write every day in the morning, when my mind was most clear and energy was at the max. Looking at the commit data from [gitstats](http://gitstats.sourceforge.net/) on the repository you can see that most of the commits were around 9-10 AM:

<img src="/images/posts/atomic-migration-strategy-for-web-teams/commits-hour-of-day.png" width="100%">

Taking a look at the same hourly view across the week you can see that later in the week I'd get started a little later, closer to 10-11 AM:

<img src="/images/posts/atomic-migration-strategy-for-web-teams/commits-hour-of-week.png" width="100%">

Other than Sunday there are basically zero after hours commits, which I guess that makes Katie and I morning people 🤷

Katie's writing style was very different from mine, she would write an entire chapter in one sitting! You wouldn't see as many commits, but you would see larger diffs in her commits. At one point we talked about if our different approaches to writing stressed each other out, her writing large occasional chunks and me with small daily changes, and we realized we were both writing in the most effective way that worked for us.

We wrote the book in an O'Reilly tool called [Atlas](https://atlas.oreilly.com/). Atlas is a web application that allows you to write rich text (and a few other formats) and is powered by GIT. It was pretty good working with Atlas, though it had a few minor quirks in the editor. We chose the HTML editor, however I wish we could have had markdown as an option. Since the book was stored in GIT you could clone the repository locally and work in whichever text editor you like and even keep a copy for yourself in Github. Overall the experience was positive and I would totally use Atlas again to write a book.

Most of the book was written from the local coffee shop [Blackberry Market](https://www.google.com/maps/place/Blackberry+Market/@41.874172,-88.0687457,17z/data=!3m1!4b1!4m5!3m4!1s0x880e53120ff84309:0x8c1fdb8c0340506b!8m2!3d41.874172!4d-88.066557). I work remotely and enjoy working the first half of the day with this view:

<img src="/images/posts/atomic-migration-strategy-for-web-teams/coffee-shop.jpg" width="100%">

## Review and Edit Process

After writing the initial copy we moved to the review and edit process. This took a couple months to get to after the writing process since the person responsible for our book was transitioning out of the company, the person who took over did an awesome job and picked things up right away. Editing took about 1 month starting in May 2018 and ending in June 2018.

This is where O'Reilly goes through your ~~code~~ copy, finds your spelling mistakes and calls you out on your grammatical errors made during moments of weakness. Actually the editors were super friendly and made wonderful suggestions that made the book so other people could read it. I am not great at spelling, anyone who looks at my source code comments can attest to this. Some of the grammatical errors I made also made me question my literacy, but hey, we made it through the process!

## Publishing

We made it to the publishing phase 🎉🎉🎉 _itshappening.gif_ (I won't actually leave the GIF here since its the most distracting thing ever created)! The book was [published as an ebook](https://www.safaribooksonline.com/library/view/atomic-migration-strategy/9781491999950/) on [Safari](https://safaribooksonline.com) and was done with the same level of quality that you'd get with a paper copy of the book. My only regret is that we did not get an animal on the cover (I would have picked wolf). If the ebook does well we _might_ get to expand on it and turn _Atomic Migration Strategy for Web Teams_ into a hard copy (and hopefully get that wolf cover). But I could not be more happy than how it all turned out, 5 years ago I could not have told you I'd ever be interested in writing a book. Now I'm already thinking about the next one 🙃

Synopsis for [_Atomic Migration Strategy for Web Teams_](https://www.safaribooksonline.com/library/view/atomic-migration-strategy/9781491999950/):

> Atomic Design, created by web designer and consultant Brad Frost, is a system for working with the fundamental building blocks—the atoms—of modern web interfaces. This guide provides hands-on instructions for stitching these simple components together to rewrite your application in a low-risk and nondisruptive way. While the ebook’s examples focus on migrating a frontend application, you can also use these techniques for mobile and backend applications.

Here's a link to the book on Safari: [_Atomic Migration Strategy for Web Teams_](https://www.safaribooksonline.com/library/view/atomic-migration-strategy/9781491999950/) we'd love a review if you get a chance to read the book. Any feedback is super helpful, especially if we get a chance to expand or release future editions!
