---
layout: post
title: "Kiwi - Updates & First demo"
date: 2014-12-08 19:50:48 +0100
comments: true
categories: [marketresearch, kiwi, demo]
footer: true
sharing: true
---

It's been more than a month since I first wrote about Kiwi. [In that post](/blog/2014/11/01/building-kiwi/) I had a "demo" section where I wrote:

>I was planning to create a screencast to demonstrate how Kiwi works and what it does but didn't get around doing it as part of this blog post. That's high on my to-do list. Another post will be on its way soon.

In the last month a lot has happened. Therefore I had to postpone the demo for a bit longer than I thought. We even had to re-evaluate one of the most essential parts of our software after realizing that our first strategy had some weaknesses that we wouldn't be able to fix simply because they were outside of our control. So what do you do when a problem is outside of your control? You find another path and bypass it. At least that's what we did but more on that after the demo.

##Demo

So first things first. Here is a ~6 minute demo below. Also read on if you're interested in the things that kept us busy in this period.

<iframe width="640" height="480" src="//www.youtube.com/embed/S1QcnZmyNpA" frameborder="0" allowfullscreen></iframe>


##TripleS vs NIPO format
When we started implementing Kiwi we thought that we could take TripleS files in order to determine the variables in a questionnaire. It was all good, a TripleS file indeed represents the structure of the questionnaire and the variables in it. But we also thought we could always parse the original NIPO data with it, without converting the data to the TripleS format before each data transfer. 

It turned out that this assumption was wrong. Later we've found out a few scenarios where it was simply impossible to predict the structure of the original data file by looking at the TripleS questionnaire structure alone. F.ex. questionnaires with jumps in the data positions or GOTO statements were resulting in undesired shifts in the variable references.

Being able to transfer data multiple times using the original NIPO data (without using any sort of conversion) is one of our design goals. This design will also enable us to completely automate this process where the user sets up the survey once and then Kiwi does the transfer continuously in the background while also displaying some sort of a dashboard to enable __live visualization__ of the data collected during the survey.

##Supporting the VAR format
To realise this design goal we've decided to add support for VAR file format. VAR file is similar to TripleS file in the sense that they both represent the structure of a questionnaire and list all the variables in a questionnaire. But unlike the TripleS format, the VAR format doesn't require the data to be converted. VAR file represents the original NIPO data. That's exactly what we want.

So now when defining surveys, Kiwi accepts both TripleS and VAR file formats. If TripleS format is used the data also needs to be converted to TripleS. For a NIPO Software client I assume this will almost never be the preferred way. The most effective way will be to use the VAR file format and transfer the original data directly without any need of conversion. This is what we enabled now.

##Background processing infrastructure
Kiwi is a web based application. Web applications are designed around quick request and responses with relatively short connection lifetimes between server and clients. However the main job of Kiwi is to process and transfer possibly very large sizes of data. This type of workload necessitates some kind of a background job processing infrastructure because it takes longer times to completely fulfill the incoming requests.

In Kiwi any data transfer request is represented as a background job, meaning that the user's request is processsed asychronously in the background while he can continue doing other things in the application. We're using a framework called [Hangfire](http://hangfire.io/) that keeps track of background jobs and manages their state transitions. Hangfire also allows us to simplify the deployment of Kiwi as it hosts the background threads right inside the web application. There is no need for a separate installation.

We want to delight our users and make Kiwi easy and fun to use. We wanted to avoid the need of manually refreshing web pages to find out if a process has finished or not. We're using [SignalR](http://signalr.net) to notify user interface about the state changes of the background processes. With this technology, users don't have to do anything manually. The page automatically updates itself in real time as the latest information becomes available.

##Performance, reliability and error handling
The main difference between a toy software and a real world software is that the former is made for demos and happy path scenarios while the latter is armed with capabilities for handling various real world inputs and conditions. Even when it fails it should fail gracefully with proper and helpful error messages, preferably pointing to some documentation on how to proceed further.

As all developers know very well, error logging is crucial for an application's success. It's one of the most important ways to diagnose the problems that are out there in production. We're using [Elmah](https://code.google.com/p/elmah/) for this purpose.

We're making extra effort to find the balance between performance and reliability. Because Kiwi will probably deal with various sizes of data from small to very large depending on the survey, it has to be able to favor reliability over performance when necessary. For those scenarios with very large input data we've been working on reducing the memory footprint and trade it off with some performance. In these scenarios loading all the data to the memory is simply not acceptable.

##Non-technical stuff
Creating a product comes with a few standard challenges.

* We're creating a company logo and a logo for Kiwi.
* We've created the first draft of our license agreement. We're now looking for a professional who can review it and help us finalize it.
* Deniz is continuously enhancing the documentation. It will be hosted on [Zendesk](https://www.zendesk.com/). Currently it's a separate entity but we want to incorporate it with the software in a way that Kiwi will know how to link to this documentation based on a given context and situation. I'm not sure if we can finish this before the first version but it's definitely on our list.


When [Pareto principle](http://en.wikipedia.org/wiki/Pareto_principle) is applied to software development, it reveals that the last %20 of the development takes as much time as the first %80. Sounds scary huh? : ) but it certainly has some truth in it. We're now very close to 100% for the first version of Kiwi. The remaining things on our todo list are non-technical, the things around actually releasing the product.

Before wrapping this up I would like to thank our employer [NIPO Software](http://www.niposoftware.com/) for their great support. We're frequently in touch and getting a lot of feedback. Really guys... Thanks a lot!

I'll keep writing as things happen. Good and bad. Transparency is the goal here.

Thank you for reading and as usual: feel free to drop a comment.

Hakan










