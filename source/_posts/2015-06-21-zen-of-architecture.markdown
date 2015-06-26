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

<br/>
####Example of Functional Decomposition

<img src="/assets/Zen_of_Architecture/IMG_2021.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2023.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2024.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2025.jpg" style="border: solid;">

##Volatility-Based Decomposition
If you would learn one thing from this entire course this should be it, Juval mentioned repeatedly. __You should always decompose a system based on volatility.__ 

* Identify areas of potential change and encapsulate them in services.
* Look for functional potential changes but __not domain functional__. Meaning that while looking for volatility, don't speculate on potential changes to the nature of the business. Don't overdo it.
* Implement behavior as interaction between services or subsystems.
* Create your milestones based on integration of these services not features.

This is the universal principle of good design says Juval. Encapsulate change to insulate. Do not resonate with change. Functional decomposition on the other hand maximizes the impact of change because it's coupled to it.
<br/>
####Challenges with Volatility-Based Decomposition
There are challenges in creating a Volatility-Based Decomposition. First of all it usually takes longer than functional because volatility is not often self evident. On the other hand features are kept thrown at your face. People around you will be feature thirsty, they'll keep asking their features like cannibals. You should instead fight the insanity and focus on the bigger picture and volatilities. Getting the management support is usually another challenge. Juval said architects should be responsible and fight against this insanity.
<br/>
####Axes of volatility
There are two axes of volatility.

* At the same customer over time
* At the same time across customers

These axes should be independent from each other. So encapsulate them from each other as well. When they're not independent it's a sign of functional decomposition.

Prior to architecture and decomposition, as part of requirements gathering and analysis, prepare a list of areas of volatility. Ask what could change along axes of volatility.
<br/>
####Example of Volatility-Based Decomposition


<img src="/assets/Zen_of_Architecture/IMG_2026.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2027.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2028.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2029.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2030.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2031.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2032.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2033.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2034.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2035.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2036.jpg" style="border: solid;">



