---
layout: post
title: "Zen of Architecture"
date: 2015-06-28 13:19:40 +0200
comments: true
categories: [workshop, devintersection, architecture]
footer: true
sharing: true
description: 
image: http://www.hakantuncer.com/assets/Zen_of_Architecture/ZenOfArchitecture.jpg
---

[I was at DevIntersection Conference](/blog/2015/05/02/devintersection-and-anglebrackets-in-phoenix/) in Phoenix around mid May 2015. Apart from the regular conference schedule, I took a few interesting workshops one of which was "Zen of Architecture" by [Juval Lowy](http://www.oreilly.com/pub/au/741).

As the name clearly implies one day workshop was all about software architecture and the ways to approach it. Juval talked about a method that he uses for decomposing systems and making design decisions.

It was a full day of content storm with as little breaks in between as possible (this is the Juval style I suppose). I've been reviewing my notes and presentation slides thinking this can become a book if I try to write everything. So instead, this post focuses on a certain part of the workshop which I think covers the core idea.

##The Method

Juval introduces a method which he calls "The Method" for decomposing a system. Achieving the right decomposition of a system is one of the most important things in software architecture. The end result of your decomposition is your architecture.

>For the beginner architect, there are many options<br/>
For the master architect, there are only a few

##Functional Decomposition
The biggest sin of any software architect. Never ever do functional decomposition!
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

Here are  a few slides from the workshop that shows Functional Decomposition in action.

<img src="/assets/Zen_of_Architecture/IMG_2021.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2023.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2024.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2025.jpg" style="border: solid;">

##Volatility-Based Decomposition
If you would learn one thing from this entire course this should be it, Juval mentioned repeatedly. __You should always decompose a system based on volatility.__ 

* Identify areas of potential change and encapsulate them in services.
* Look for functional potential changes but __not domain functional__. Meaning that while looking for volatility, don't speculate on potential changes to the nature of the business. Don't overdo it.
* Implement behavior as interactions between services or subsystems.
* Create your milestones based on integration of these services not features.

This is the universal principle of good design says Juval. Encapsulate change to insulate. Do not resonate with change. Functional decomposition on the other hand maximizes the impact of change because it's coupled to it.
<br/>
####Challenges with Volatility-Based Decomposition
There are challenges in creating a Volatility-Based Decomposition. First of all it usually takes longer than functional because volatility is not often self evident. On the other hand features are kept thrown at your face. People around you are feature thirsty, they'll keep asking for their features. You should instead fight the insanity and focus on the bigger picture and volatilities. Getting the management support is usually another challenge. Juval said architects should be responsible, fight against these opposing forces and do what is right.
<br/>
####Axes of volatility
There are two axes of volatility.

* At the same customer over time
* At the same time across customers

These axes should be independent from each other. So encapsulate them from each other as well. When they're not independent it's a sign of functional decomposition.

Prior to architecture and decomposition, as part of requirements gathering and analysis, prepare a list of areas of volatility. Ask what could change along the axes of volatility.
<br/>
####Example of Volatility-Based Decomposition

Below, you'll find 5 slides from the workshop that brainstorms on possible volatilities of a trading system. Here are some guidelines for capturing volatility:

* The objective is to have a mindset of "what could possibly change?"
* Capturing the areas of volatility earlier is better than later. The later you figure it out the more it will cost you.
* Once settled on the ares of volatility encapsulate them in components of architecture.
* You don't need an exhaustive list. This is a process of diminishing returns. Don't overdo it.
* Some volatile areas may relate too much to the nature of your business. This type of volatility is out of your scope.

<img src="/assets/Zen_of_Architecture/IMG_2026.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2027.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2028.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2029.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2030.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2032.jpg" style="border: solid;">
<br/>
####Let's analyze this decomposition.

Transition from list of areas of volatility to services is hardly ever pure 1:1. Sometimes a single service encapsulates multiple areas. Some areas may map to an operational concept or may be encapsulated in a third party component.

Always encapsulate the data storage volatility behind data access services. Encapsulate where the storage is, what technology is used to access it and refer your storage as storage, not as database or whatever the actual technology is.

Following 3 slides do further analysis of the decomposition in this example. Please refer to the diagram above as you read the key points:

<img src="/assets/Zen_of_Architecture/IMG_2034.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2035.jpg" style="border: solid;">
<img src="/assets/Zen_of_Architecture/IMG_2036.jpg" style="border: solid;">

##Decomposition and Business

Avoid encapsulating changes to the nature of your business. Because

* you'll need it very rarely
* you'll be diving into speculation and speculation based design trap
* when you do it you'll probably do it very poorly because there are too many unknowns (and again speculations)

While designing a system for your business don't only focus on your own also keep your competitor in mind. Design both for you and your competitor. This is a useful posture in designing systems and it's not about features or functionality but it's about understanding the nature of the business. Keeping your competitors in mind and not only focusing on your own business will help you understand the nature of the business even better. This way you can have a better judgement about what is volatile and what is not.

##Decomposition and Longevity

Volatility is very closely tied to longevity. The longer things do not change the longer they have till they do change or are replaced. The more frequently things change the more likely they would change in the future.

You must take into account impact from change regardless of your requirements. Ask yourself what has changed over the past 5-7 years and what will change in the next 5-7 years. Encapsulate things that would change within the life of your system.

##Further topics

For the rest of the day Juval dived into 

* layered architectures
* typical layers
* definition of managers, engines, resource access services and differences between them
* open and closed architectures
* what not to do when using "the method"
* creating call graphs within system components and observing/revealing anti-patterns
* a bonus section called "what about agile?"

All in all it was a mind tickling workshop and I hope I could give you more than only a taste. I may write other posts about the bullet points above if/when I find the time.

Thanks for reading

Hakan


