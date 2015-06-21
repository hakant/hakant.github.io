---
layout: post
title: "Zen of Architecture"
date: 2015-06-21 12:19:40 +0200
comments: true
categories: [workshop, devintersection, architecture]
footer: true
sharing: true
description: 
image: http://www.hakantuncer.com/assets/AngleBrackets_DevIntersection/anglebrackets_main.png
---

[I was at the DevIntersection Conference](/blog/2015/05/02/devintersection-and-anglebrackets-in-phoenix/) in Phoenix around mid May 2015. Apart from the conference schedule, I took a few interesting workshops one of which was "Zen of Architecture" given by [Juval Lowy](http://www.oreilly.com/pub/au/741).

As the name clearly implies the one day workshop was all about software architecture and the ways to approach it. Juval talked about a method that he uses for decomposing a system and making design decisions.

It was a full day of content bombardment with as little breaks in between as possible (this is the Juval style I suppose). I've been reviewing my notes and the presentation slides thinking this could be a long blog post. I'll try to summarize the bits that I find important and most interesting.

##The Method

Juval introduces a method which he calls "The Method" for decomposing a system. Achieving the right decomposition of a system is one of the most important things in software architecture. Basically the end result of your decomposition is your architecture.

>For the beginner architect, there are many options<br/>
For the master architect, there are only a few

##Functional Decomposition
The biggest sin of any software architect. Never ever do functional decomposition again!
<br/>

####What is it?

* This is also known as the flow-chart decomposition.
* Basing services and system components on the order of logical steps in use cases.
* Decomposing the system based on features, functional requirements and time.

Juval emotionally talks about "functional decomposition" easily for more than an hour. Can't repeat enough how bad it is and how we're all guilty of doing it from time to time and why we should be very conscious about resisting our bad habits. Say "functional decomposition" to Juval one more time and he'll kill you right there without blinking.
<br/>

####Why is it so bad?

* It leads to duplicating behaviors across services.
* It leads to explosion and bloating of services and intricate relationships inside and between them.
* It couples multiple services to data contract.
* Promotes implementing use cases in higher level terms thus difficult to reuse same behavior in another use case.
* Couples services to order and current use cases.
* Prevents single point of entry.





