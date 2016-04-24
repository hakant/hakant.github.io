---
layout: post
title: "Implementing an App Walkthrough"
date: 2016-04-20 22:23:47 +0200
comments: true
categories: [hackathon, aurelia, javascript, ux]
footer: true
sharing: true
description: Implementing an App Walkthrough for Hackathon Planner
image: http://www.hakantuncer.com/assets/App_Walkthrough/Hackathon-Planner-Walkthrough.gif 
---

I think most professionals who design product interfaces (both physical or software interfaces) will agree 
that simplicity and discoverability of an interface is a big winning factor. Simple and discoverable 
interfaces are usually self explanatory and for this very reason they tend to provide great user 
experiences.

On the other hand, this doesn't mean we should stop providing documentation or user manuals for our 
products. A discoverable user interface with a clean, easily accessible documentation is I think what
every product deserves.

When it comes to user manuals or guides, I'm so much in favor of a user manual that's deeply connected or 
embedded to the user interface rather than a pile of content parked somewhere completely disconnected from 
the actual product.

UI embedded manuals can be designed in many ways, one of which is through a concept called "App Walkthrough". 

##App Walkthrough

From a conceptual point of view I knew what an app walkthrough is for a very long time but before I sat down
to write this post I was tempted to call it "in-app tutorials". After some searching around the topic, I 
discovered that it actually has a [common name](http://www.dtelepathy.com/blog/design/ux-flows-how-when-to-design-app-walkthrough).

>A walkthrough is the process of intentionally revealing functionality to a user.

Implementing a successful app walkthrough is [not an easy endeavour at all](https://www.nngroup.com/articles/mobile-instructional-overlay/).
It's like a double edged sword. If you [get it right](http://www.webdesignerdepot.com/2014/07/how-to-design-a-successful-web-app-walkthrough/) 
it's all unicorns and butterflies. If you get it wrong you can do more harm than good.

## Implementing one for Hackathon Planner

A couple of days ago I started flirting with the idea of implementing a walkthough for the [Hackathon 
Planner](https://github.com/hakant/HackathonPlanner). This is what came out:

![Hackathon Planner Walkthrough](/assets/App_Walkthrough/Hackathon-Planner-Walkthrough.gif)

You can try it by visiting the [Hackathon Planner preview website](http://hackathonplanner.s3-website.eu-central-1.amazonaws.com/).
It is designed to be viewed only once and then gets out of the way forever. There are two things that I still 
want to add to this:
 
* A quick opt out button on each individual step of the walkthough so that users have an easy way to exit 
the tutorial without being forced to complete it.


>__Update:__ Right after I wrote this blog post, I came across a tutorial that's built into GitHub Desktop (for
>Mac). See the screenshot below. Clicking "Got it!" takes the user to the next step of the tutorial while
>clicking the close button on the top left corner stops the tutorial. It's brilliant.
><br/><br/>
>![GitHub Desktop Walkthrough](/assets/App_Walkthrough/GitHub_Desktop_Walkthrough.png)

* An overlay that makes the UI frozen while the tooltips are running. If a user doesn't follow the 
tooltip popovers and starts clicking around, things get messy (and this is bound to happen if I 
know anything about users).

## Show me the code

Inside the Aurelia app, I've created a component called "_TooltipService_" which is responsible for 
orchestrating the walkthrough. There's also a separate data holder class "_Tooltips_" that declaratively defines all walkthrough 
steps and their contents, positions etc.

Tooltips are created using the [popover component from Bootstrap 4](http://v4-alpha.getbootstrap.com/components/popovers/) 
(still in alpha at the time of this writing).

<br/>
<script src="https://gist.github.com/hakant/48a5b7c5a4e4f34102f393fbd80dc783.js"></script>

<br/>
<script src="https://gist.github.com/hakant/36345d12467d304e29e444430a8bf747.js"></script> 

~Hakan

  