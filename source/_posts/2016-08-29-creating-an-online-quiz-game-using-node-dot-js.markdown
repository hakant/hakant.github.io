---
layout: post
title: "Creating an online quiz game using node.js, hyperdev, play station buzz controllers, raspberry PI and Nfield"
date: 2016-08-29 20:58:34 +0200
comments: true
categories: node.js, hyperdev, nfield
---

I've been writing about Hackathons and the planner application I've been busy 
with for a while now. The good news is that [Hackathon Planner](https://github.com/hakant/HackathonPlanner) has been live for a while and used by 
[NIPO Software](http://niposoftware.com/) for the last hackathon event on 25 August 2016. There have been 36 hackathon ideas
created, voted and team compositions have been made using the software. You can see it in action 
<a href="/assets/Online_Quiz_Game/HackathonPlanner.gif" target="_blank">over here.</a>

But this post is about something else. Something much crazier than most hackathon ideas. It is the most popular
idea of the day from my colleague Rutger de Jong: Converting [Nfield](https://www.niposoftware.com/Products/Nfield) into an online 
quiz game that can be controlled by play station buzz controllers.

![Quiz Game with Play Station Buzz](/assets/Online_Quiz_Game/PlayStationBuzz.jpg)

## The Architecture

![Quiz Game - Architecture Diagram](/assets/Online_Quiz_Game/Architecture-Diagram.png)

1. Hyperdev.com (node.js)

[Hyperdev](https://hyperdev.com) is a prototyping environment where you can start up and develop a simple node.js application 
right within the browser and see your application up and running live in a matter of seconds. Every time you save a file 
in your project, hyperdev deploys it automatically with an amazing speed. It uses docker containers and AWS behind the scenes. 
It even supports https out of the box, which was important for our purposes.

This node.js app running on hyperdev is our orchestrator. It's basically responsible with broadcasting incoming websocket
messages to all subscribers. All pieces of the puzzle (buzz controllers, raspberry pi and nfield) talk to the app on 
hyperdev instead of directly talking to each other (which would require a knowledge of where they are in the network).

2. node.js app on Mac











