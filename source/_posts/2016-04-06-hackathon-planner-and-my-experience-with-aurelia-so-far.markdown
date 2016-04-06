---
layout: post
title: "Hackathon Planner &amp; My experience with Aurelia so far"
date: 2016-04-06 02:00:00 +0200
comments: true
categories: [hackathon, aurelia, javascript, aws]
footer: true
sharing: true
description: A first look at the Hackathon Planner
image: http://www.hakantuncer.com/assets/First_Look_at_Hackathon_Planner/Hackathon_Planner.png
---

~2 months ago [I wrote about Hackathon Planner](/blog/2016/01/30/hackathon-planner-for-nipo-software/)
and hustled the idea as a candidate project for the next hackathon at NIPO Software. Then the hackathon day arrived. 
We formed a team of 6 and spent a day working on it. Rutger shot a time lapse video from that day.
<br/>

<iframe width="700" height="393" src="https://www.youtube.com/embed/GMK8vvJm87A" frameborder="0" allowfullscreen></iframe>

Since then, I continue working on the Hackathon Planner whenever I find some free time here and there. Rutger also 
occasionally shows up in the commit history contributing with his CSS superpowers.

I use [Azure](https://azure.microsoft.com/en-us/) everyday at work and this time I want to experiement with 
[AWS](https://aws.amazon.com/). I've put up an [early preview of Hackathon Planner](http://hackathonplanner.s3-website.eu-central-1.amazonaws.com/) 
on Amazon S3. I'll keep updating it from this point on. At the time of this writing, it's only a client 
side preview - actions are only saved on the local storage of the browser. Not very useful yet but it soon will be. 
[Check it out.](http://hackathonplanner.s3-website.eu-central-1.amazonaws.com/)

## Aurelia

[Aurelia](http://aurelia.io/) has been my framework of choice for this 
[single page application](https://en.wikipedia.org/wiki/Single-page_application). It truly is a great framework in
the sense that it really gets out of your way. After I learned some of the fundamentals of Aurelia, I was basically
good to go. Some periods I even forgot that I was using a framework because it is simply not in your face. Let me 
show you some code samples from the project.
<br/>

#### ES6 and ES7 in action
Aurelia takes advantage of the latest javascript standards as much as possible. Take a look at this "nav-bar" 
component from the Hackathon Planner.  
  
<br/>
<script src="https://gist.github.com/hakant/27562e37df6d7bf59f24fd12b316045a.js"></script>

* First three lines are an example of [ES6 module importing](https://developer.mozilla.org/en/docs/web/javascript/reference/statements/import). 
This is how a javascript module can make use of code from external libraries.
* Line #5 is an example of an [ES7 Decorator](https://medium.com/google-developers/exploring-es7-decorators-76ecb65fb841#.5obmn2df6) that is in this case 
[used by Aurelia for dependency injection](http://eisenbergeffect.bluespire.com/aurelia-update-with-decorators-ie9-and-more/).
Basically we're telling Aurelia to pass those components as constructor arguments to the NavBar component.
* And the rest of the code is the NavBar class that is written based on the [ES6 class specification](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes).
See how clean this component is - it's pure javascript.
<br/>  

#### Dynamic application root
Code below is the entry point to the Hackathon Planner: "main.js".
  
<br/>
<script src="https://gist.github.com/hakant/8c94080e4c61ea04bfaf1a009dbcc25b.js"></script>

* Line #2 imports a custom authentication service that I wrote and it gets resolved as a singleton instance at line #13.
In Aurelia all dependencies are registered as singleton by default (unless you specify it otherwise). 
* Based on the authentication status (line #14) aurelia app root is set either to the app or to the login 
component. The ability to set arbitrary application roots is a powerful feature of Aurelia that enables some 
[interesting scenarios](http://patrickwalters.net/creating-multipage-apps-using-aurelia-2/).
* I borrowed this idea from [Matthew James Davis](http://davismj.me/blog/aurelia-auth-pt2/).
<br/>

#### Validation
Aurelia also comes with a [validation plugin](https://github.com/aurelia/validation) and [it's currently undergoing some big refactoring](http://blog.durandal.io/2016/03/23/aurelia-babel-6-and-jspm-update/).
It still wouldn't hurt to show how it looks like at the moment. Note that the @ensure decorator is recently 
deprecated (which means I'll also need to adapt this code going forward).

This is a stripped down version of the project class in Hackathon Planner:

<br/>
<script src="https://gist.github.com/hakant/69363eaf32e2052d2d2697335704555e.js"></script>

* Line #8 and #10 sets validation rules on those properties and line #20 initializes the validation for this 
object.

And this event handler in "new-card.js" validates a project before adding or updating it:

<br/>
<script src="https://gist.github.com/hakant/bbb87af2f2306724bbd1981973b8d83f.js"></script>
<br/>

#### Beautiful components

In Aurelia, user interface components are composed of [view and view-model pairs](http://aurelia.io/docs.html#/aurelia/framework/1.0.0-beta.1.1.4/doc/article/creating-components).
These two are created by Aurelia's dependency injection framework and gets linked to each other via data binding.

Here is one of those pairs from the Hackathon Planner. This is how the overview page is rendered:

<br/>
<script src="https://gist.github.com/hakant/6ab1a376f50b80bd115f765e16a551f4.js"></script>

<br/>
<script src="https://gist.github.com/hakant/7296309c6a7166254ac701a6b27caf47.js"></script> 

* Notice how a component is required into a template at line #2. 
* And then all projects are looped through and each one is rendered into another component called "idea-card" 
at line #5.

Thank you for reading and make sure you check out the [Hackathon Planner on GitHub](https://github.com/hakant/HackathonPlanner) 
and [its early preview deployment](http://hackathonplanner.s3-website.eu-central-1.amazonaws.com/).

~Hakan 



 