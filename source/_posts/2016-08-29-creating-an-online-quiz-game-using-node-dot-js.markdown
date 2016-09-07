---
layout: post
title: "Creating an online quiz game using Node.js, Hyperdev, PlayStation Buzz controllers, Raspberry PI and Nfield"
date: 2016-09-07 20:58:34 +0200
comments: true
categories: [node.js,hyperdev,nfield]
footer: true
sharing: true
description: How to convert Nfield into an online quiz game
image: http://www.hakantuncer.com/assets/Online_Quiz_Game/Architecture-Diagram.png
---

I've been writing about hackathons and the planner application I've been busy 
with for a while now. The news is that [Hackathon Planner](https://github.com/hakant/HackathonPlanner) is live 
and used during the last hackathon event on 25 August 2016 at [NIPO Software](http://niposoftware.com/). 
36 hackathon ideas were created, voted and team compositions were made using the software. You can 
<a href="/assets/Online_Quiz_Game/HackathonPlanner_Final.gif" target="_blank">see it in action here</a>.

This post however, is about something else. It's about a crazy idea of my colleague Rutger de Jong and how we 
brought it to life during that last hackathon day: Converting [Nfield](https://www.niposoftware.com/Products/Nfield) into an online quiz game that 
can be controlled by PlayStation Buzz controllers.

![Quiz Game with Play Station Buzz](/assets/Online_Quiz_Game/PlayStationBuzz.jpg)

## The Team

Rutger brought together five talented people from NIPO Software with diverse skills. Then he was scratching 
his head (on the left) thinking "what have I done..." : ) 

![NIPO Software Hackathon - Team](/assets/Online_Quiz_Game/team.jpg)

## The Architecture

![Quiz Game - Architecture Diagram](/assets/Online_Quiz_Game/Architecture-Diagram.png)

## How it works
<br/>

* A shared screen with a browser is opened and an online interview is started by visiting a specific URL to Nfield. 
This online interview is running our little quiz.
* Each player grabs one of the PlayStation Buzz controllers that are connected to a computer runing 
a node.js application to interpret inputs coming from the controllers.
* The online interview that was started in the browser is actually a quiz with a special template (CSS & Javascript).
* A random sequence of questions start appearing on the screen, each for 10 seconds.
* Players give their answers to each question simultaneously. These answers are received by a node.js application 
listening inputs from Buzz controllers and pushes events to the app running on Hyperdev.
* Application running on Hyperdev acts as a message hub and broadcaster. It relays the incoming messages to the quiz 
running on Nfield and also to the Raspberry PI.
* The quiz, running on the browser, receives these message (via our customized template) and picks the corresponding 
answers selected by players.
* As time is ticking away for each question, the customized template that's running the quiz emits events to the 
Raspberry PI, telling it to buzz and blink harder.
* Scores are continuously calculated as quiz progresses and in the end the winner is announced.

## The code

You can find all the code that powers this project in the following repository on GitHub:
https://github.com/hakant/NfieldQuizGame

## Fun at the demo

The most obvious (and fun!) demo for a quiz game is to find a number of volunteers from the audience and run a contest!
So that's what we did : )

![Hackathon - Demo](/assets/Online_Quiz_Game/demo.jpg)

... and here is a screenshot from the quiz:

![Hackathon - Nfield Quiz - Screenshot](/assets/Online_Quiz_Game/Quiz-Screenshot.png)

All in all, once again, it was a great event at the NIPO Software office in Amsterdam. Who knows.. maybe in the very near future, 
this little fun project can be put into actual use during various market research events. Without a doubt, it showcases the 
power of Nfield and how customizable it is for variety of scenarios.

If you want to know more about each component and what it specifically does, read on.

## System Components
<br/>

#### Hyperdev.com / node.js

[Hyperdev](https://hyperdev.com) is a prototyping environment where you can start up and develop a simple node.js application 
right within the browser and see your application up and running live in a matter of seconds. Every time you save a file 
in your project, hyperdev deploys it automatically in an amazing speed. It uses docker containers and AWS behind the scenes. 
It also supports https out of the box, which was important for our purposes (Nfield services run on https so should any
integration or extension with it).

This node.js app running on hyperdev is our orchestrator. It is basically responsible with broadcasting incoming websocket
messages to all subscribers. Each piece of the puzzle (buzz controllers, raspberry pi and nfield) talk to this node app on 
hyperdev instead of directly talking to each other, which btw would complicate things as each application would then have to 
discover where the others are and be able to connect to them somehow. 

In the architecture above, as long as everyone can reach hyperdev, successful communication is guaranteed.

<br/>
#### Mac with Buzz Controllers / node.js

While PlayStation buzz controllers are perfect for a quiz game, the immediate question for our little hackathon 
project was; how do we connect to and control these things? 

[Node.js is everywhere](https://www.youtube.com/watch?v=Qkkmz5VZoMQ), it really is. There is a node package called 
[node-hid](https://github.com/node-hid/node-hid) which allows you to access USB Human Interface Devices (HID) from 
node.js. It basically allows you to read from and write to these devices by using node buffers. We could even connect
these buzzers to Raspberry PI but we ran into a few security issues and pivoted to using a mac instead.

Of course, just being able to read the controller's input buffer doesn't automatically mean you know how to interpret it.
It speaks a certain protocol which you have to understand. While scratching our heads, we found this article: 
[Working with USB devices in .NET and C#](http://www.developerfusion.com/article/84338/making-usb-c-friendly/) where 
the author describes the protocol clearly, well somewhat clearly:

![PlayStation Buzz - Protocol](/assets/Online_Quiz_Game/buzz-protocol.jpg)

Long story short, each controller has a reserved space on a 6 byte area, inside which they flip bits on and off based
on the state of their buttons.

Here is a section of the node.js code running on the Mac that reads inputs from buzz controllers:

<br/>
<script src="https://gist.github.com/hakant/7ee37ee00fccfbf487d2c6d3328f13dd.js"></script>

<br/>
#### Raspberry PI / node.js

Since we couldn't succeed in reading inputs from PlayStation Buzz controllers on our Raspberry PI, we had to come up with another very useful
role for it - because we simply can't imagine a hackathon project without the PI (!). 

One of our teammates said "let's use it to signal remaining time at each question - with some touch of tension".
Then we went ahead and placed a bunch of lights on the PI and a loud buzzer. Using node.js and socket.io, it was 
listening to certain type of events to start buzzing with low and high frequencies. As contestants were running out of 
time while answering each question, the frequency of buzz and light effects were increasing. 

<br/>
#### Nfield Online Interviewing System / .NET, Azure

Nfield Online is a data collection platform powered by NIPO Software. A SaaS platform where market research companies
can create surveys and reach their respondents. Nfield Online is bundled with a powerful domain specific
language, ODIN, created for scripting simple or complex questionnaires. Nfield also has a powerful templating system 
where customers can create and run their own templates in their surveys (both for changing the look & feel and behavior of 
their questionnaires).

We used this templating system (HTML, CSS and Javascript) to tap into a running interview (in this case, our quiz) and 
control it via PlayStation Buzz controllers over the internet. We also used it to send messages to the Raspberry PI
to indicate remaining time per question.

When this template is activated on the interview, which is running our quiz, it connects to the socket.io server
on hyperdev and starts listening to "button pressed" events from PlayStation Buzz controllers. When a button 
is pressed it receives the event and selects the corresponding answer on the quiz for that contestant.

I had a lot of fun working on this project and also writing about it in this blog. Every
technology company should make hackathons a part of their culture. The benefits might not be immediately clear
in the beginning but all those doubts disappear after a few events.

<br/>
Thank you for reading.

~ Hakan