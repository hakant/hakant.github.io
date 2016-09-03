---
layout: post
title: "Creating an online quiz game using node.js, hyperdev, PlayStation buzz controllers, raspberry PI and Nfield"
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
quiz game that can be controlled by PlayStation buzz controllers.

![Quiz Game with Play Station Buzz](/assets/Online_Quiz_Game/PlayStationBuzz.jpg)

## The Architecture

![Quiz Game - Architecture Diagram](/assets/Online_Quiz_Game/Architecture-Diagram.png)

#### Hyperdev.com (node.js)

[Hyperdev](https://hyperdev.com) is a prototyping environment where you can start up and develop a simple node.js application 
right within the browser and see your application up and running live in a matter of seconds. Every time you save a file 
in your project, hyperdev deploys it automatically in an amazing speed. It uses docker containers and AWS behind the scenes. 
It also supports https out of the box, which was important for our purposes.

This node.js app running on hyperdev is our orchestrator. It's basically responsible with broadcasting incoming websocket
messages to all subscribers. Each piece of the puzzle (buzz controllers, raspberry pi and nfield) talk to the node app on 
hyperdev instead of directly talking to each other (which would require a knowledge of where they are in the network).

<br/>
#### node.js app on Mac

While the PlayStation buzz controllers are perfect for a quiz game, the immediate question for our little hackathon 
project was: "how do we connect and control this thing?". 

[Node.js is everywhere](https://www.youtube.com/watch?v=Qkkmz5VZoMQ), it really is. There is a node package called 
[node-hid](https://github.com/node-hid/node-hid) which allows you to access USB Human Interface Devices (HID) from 
node.js. It basically allows you to read from and write to these devices by using node buffers.

Of course, just being able to read the controller's input buffer doesn't automatically mean you know how to interpret it.
It speaks a certain protocol which you have to understand. While scratching our heads, we found this article: 
[Working with USB devices in .NET and C#](http://www.developerfusion.com/article/84338/making-usb-c-friendly/) where 
the author describes the protocol somewhat clearly:

![PlayStation Buzz - Protocol](/assets/Online_Quiz_Game/buzz-protocol.jpg)

Here is the pseudo node.js code running on the mac that reads inputs from buzz controllers:

TODO: Understand why these code styles are gone!

``` javascript

var HID = require('node-hid');
var bitwise = require('bitwise');
var socket = require('socket.io-client')('https://frill-crafter.hyperdev.space');

device.on("data", function(data) {
    // Interpret data buffer and figure out which buttons on which controllers are being pressed
    // ... (there's a link to the actual code on github at the end of this post)

    // Then emit the following message to the socket.io server running on hyperdev (per controller)
    socket.emit('button-clicked', `{ "ControllerId": ${controllerId}, "ButtonId": ${buttonId} }`);
}


```












