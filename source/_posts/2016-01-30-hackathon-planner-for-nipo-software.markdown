---
layout: post
title: "Hackathon Planner for <br/> { NIPO Software }"
date: 2016-01-30 09:29:33 +0100
comments: true
categories: [hackathon, aurelia, nodejs, aws]
footer: true
sharing: true
description: Building an open source hackathon planner
image:
---

I learn by reading and practicing. When I only read and nothing else (i.e no practicing, no note taking), I almost completely forget what I learned after a month or two. I think most people are wired this way. Think about a book you've read 6 months ago and see how much of it you can still remember. It's scary.

Learning something new in programming is no different. Maybe even worse. But there's good news, because in programming [learning by practicing](http://jamesclear.com/learning-vs-practicing) is very easy. You just need a computer and that's it! And hey, now we're in the cloud era. If you need more, go and get what you need with a couple of clicks! It's crazy how much programmers can do these days while sitting on a couch with a cup of coffee. You just need the passion. The rest is simply there available for you to grab.

## Hackathons

[Hackathons](https://en.wikipedia.org/wiki/Hackathon) are a great way of [learning by doing](http://news.uchicago.edu/article/2015/04/29/learning-doing-helps-students-perform-better-science) and we at [NIPO Software](http://niposoftware.com/) have started making this a company tradition for the last year or so, thanks to my colleague [Qa'id Jacobs](https://twitter.com/qaidj) who took the first initiative and still putting a lot of effort into organizing them.

* TODO: Take a good photo from one of our hackatons

One of the ideas during previous hackathon (summer 2015) was building a "Hackathon Planner". That time around it didn't get enough attention, so was never picked up. Currently all ideas and team compositions are kept and maintained in an Excel file, which is the best and the worst tool of all times in computing history. So how about building a planner that can help people capturing ideas and forming teams around those ideas in a fun way.

## A Hackathon Planner - [MVP](https://en.wikipedia.org/wiki/Minimum_viable_product)

A minimum viable hackathon planner;

* Should be open source (not only [NIPO Software](http://niposoftware.com/), anyone should be able to make use of it).
* Should allow capturing of ideas in a modern way (I hear [SPAs](https://en.wikipedia.org/wiki/Single-page_application), [markdown](https://en.wikipedia.org/wiki/Markdown), HTML dialogs, subtle animations.. anyone?).
* Should allow simple ways of reacting to those ideas (i.e sending likes and later on comments).
* Should be simple to sign up and login (GitHub integration?).
* Should allow simple team formation (mark the idea that you want to work for).
* Monitoring live as teams form and evolve. Just an excuse to use something like signalR or socket.io.
No wait, actually it makes the team formation process much more exciting to watch. Watching as ideas live or die.

## Ingredients

At the beginning of this post I talked about practicing and how it's crucial for learning new programming skills. I'm very much into [Aurelia](http://aurelia.io/) and [node.js](https://nodejs.org/) these days. Sure I'll throw in a NoSQL database too and see how much of these decisions will hurt during production.

I've seen [@jbogard](https://twitter.com/jbogard/) putting this nicely:

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">one thing i try very hard to do is to not promote any tool/language/framework/platform/pattern until i&#39;ve had to support it in production</p>&mdash; Jimmy Bogard (@jbogard) <a href="https://twitter.com/jbogard/status/691992462483677184">January 26, 2016</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

My recent theory is: __how cool a software platform, framework, database or a language is directly proportional to the amount of pain one will get during production support__.

So I'll see how this one goes with my choice of technology stack. If I can stop slacking and get to finish the MVP of course ;)

~Hakan
