---
layout: post
title: "Hackathon Planner for <br/> { NIPO Software }"
date: 2016-01-30 09:29:33 +0100
comments: true
categories: [hackathon, aurelia, nodejs]
footer: true
sharing: true
description: Building an open source hackathon planner
image:
---

I learn best by reading and practicing. When I only read (i.e no practicing, no note taking), I almost completely forget what I learned after a month or two. I think most people are wired this way. Think about a book you read 6 months ago and see how much of it you can still remember. It's scary.

Learning something new in programming is no different. Maybe even worse. But there's good news, because in programming [learning by practicing](http://jamesclear.com/learning-vs-practicing) is very easy. You just need a computer and that's it! And hey, now we're in the cloud era. If you need more, go and get what you need with a couple of clicks! It's crazy how much programmers can do these days while sitting on a couch with a cup of coffee. You just need the passion. The rest is simply there available for you to grab.

## Hackathons

[Hackathons](https://en.wikipedia.org/wiki/Hackathon) are a great way of [learning by doing](http://news.uchicago.edu/article/2015/04/29/learning-doing-helps-students-perform-better-science) and we at [NIPO Software](http://niposoftware.com/) have started making this a company tradition for a year or so, thanks to my colleague [Qa'id Jacobs](https://twitter.com/qaidj) who took the first initiative and continues putting effort into organizing them. Below are some pictures from our last hackathon around summer of 2015.

![NIPO Software Summer 2015 Hackathon 1](/assets/Hackathon_Planner/Hackathon_NIPO_1.jpg)

![NIPO Software Summer 2015 Hackathon 2](/assets/Hackathon_Planner/Hackathon_NIPO_2.jpg)

![NIPO Software Summer 2015 Hackathon 3](/assets/Hackathon_Planner/Hackathon_NIPO_3.jpg)

One of the ideas during this hackathon was building a "Hackathon Planner". Didn't get enough love, so was never picked up. Currently all ideas and team compositions are kept and maintained in an Excel file - the best and the worst tool of all times in computing history. So how about building a planner that can help people capturing ideas and forming teams around those ideas in a fun way?

## A Hackathon Planner - [MVP](https://en.wikipedia.org/wiki/Minimum_viable_product)

A minimum viable hackathon planner;

* should be open source (not only [NIPO Software](http://niposoftware.com/), anyone or any company should be able to make use of it if they want to do so).
* should allow capturing of ideas in a modern way (I hear [SPAs](https://en.wikipedia.org/wiki/Single-page_application), [Markdown](https://en.wikipedia.org/wiki/Markdown), subtle animations, web sockets, responsive design).
* should allow simple ways of reacting to those ideas (i.e sending likes and possibly comments in the future).
* should be simple to sign up and login (built in GitHub authentication?).
* should allow a simple way of forming teams (mark the idea/team that you want to join and done!).
* should allow live monitoring of teams as they form and evolve. It makes the team formation process much more exciting to follow. Watching as ideas live and die. Well.. OK I'll be honest with you, actually it's just an excuse to use something like [signalR](http://www.asp.net/signalr) or [socket.io](http://socket.io/).

Based on these thoughts above I've created a very [basic clickable mockup](https://hackathonplanner.mybalsamiq.com/projects/hackathonplanner/prototype/Hackathon+Ideas?key=461d3fd4697bd193977050ca80bdbe84a1383d22) that can give a general idea and show some of the basic interactions:

[![Hackathon Planner - Mockup](https://hackathonplanner.mybalsamiq.com/mockups/4089623.png)](https://hackathonplanner.mybalsamiq.com/projects/hackathonplanner/prototype/Hackathon+Ideas?key=461d3fd4697bd193977050ca80bdbe84a1383d22)

## Ingredients

I'm very much into [Aurelia](http://aurelia.io/) and [node.js](https://nodejs.org/) these days. So that's what I'll be using. Sure I'll throw in a NoSQL database too and see how much these decisions will hurt me during production.

I saw [@jbogard](https://twitter.com/jbogard/) putting it nicely:

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">one thing i try very hard to do is to not promote any tool/language/framework/platform/pattern until i&#39;ve had to support it in production</p>&mdash; Jimmy Bogard (@jbogard) <a href="https://twitter.com/jbogard/status/691992462483677184">January 26, 2016</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

I recently have this theory: How cool a platform, framework, database or a language is directly proportional to the amount of pain one will get during production support.

So we’ll see how this one goes with the choices that I made. If I can stop slacking off and get to finish the MVP of course.

~Hakan
