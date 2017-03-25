---
layout: post
title: "Setting up Continuous Delivery for a Node.js REST API - Part 2"
date: 2017-03-25 09:48:02 +0100
comments: true
categories: [architecture,testing,CI]
footer: true
sharing: true
description: A few months back I decided to convert my hobby project - Hackathon Planner API - from pure Javascript to TypeScript. This time I sat down to build an Automated Test Suite and a Continuous Delivery pipeline around it.
image: http://www.hakantuncer.com/assets/ContinuousIntegration_Part1/CI.jpg
---

## Introduction

[In the first part](/blog/2017/02/17/setting-up-continuous-integration-and-delivery-for-a-nodejs-rest-api/) of this blog post, I shared some fundamental ideas that form as background information for what I want to achieve here. So if you haven’t read that one yet, I recommend you to [check it out](/blog/2017/02/17/setting-up-continuous-integration-and-delivery-for-a-nodejs-rest-api/).

A few months back, I decided to convert my [hobby project — Hackathon Planner API](https://github.com/hakant/HackathonPlannerAPI) from pure Javascript to TypeScript and [wrote a blog post about it](/blog/2016/11/08/converting-a-node-dot-js-express-api-to-typescript/). This time I sat down to build an Automated Test Suite and a [Continuous Delivery (CD)](https://www.martinfowler.com/articles/continuousIntegration.html) pipeline around it.

Effective automated testing is a natural prerequisite for Continuous Integration & Delivery. How can it not be? If you’re not getting quick and broad feedback from your software, how can you delivery frequently? So having a testable architecture and an effective test strategy is very crucial. Let’s dive in.

## Right Architecture for Subcutaneous Testing

I’m a huge fan of [subcutaneous testing](https://martinfowler.com/bliki/SubcutaneousTest.html). This type of testing starts right under the UI layer with a large scope (which preferably spans all the way down to the data store) can have a great return on investment. On [Kent Beck’s feedback chart](https://m.facebook.com/notes/kent-beck/making-making-manifesto/857477870951745/) it would score high up and to the right: __fast and broad feedback__.

![Kent Beck's Feedback Chart](/assets/ContinuousIntegration_Part2/Kent_Beck_Feedback_Chart.png)

So now that we want to target our automated tests below the delivery mechanism layer (UI, Network etc.), this is where an important architectural decision comes into question. What is the main boundary of my application? Where does my significant business logic start? Plus, how can I make my application boundary very visible and clear to all developers? If you follow down this path of thinking, one nice place to end up is a combination of [command](https://en.wikipedia.org/wiki/Command_pattern) and [mediator](https://en.wikipedia.org/wiki/Mediator_pattern) patterns.

The combination of these patterns is about sending commands down through a very narrow facade, where every command has a very clean input and output “[data transfer objects](https://en.wikipedia.org/wiki/Data_transfer_object)” or POCO’s, POJO’s.. whatever you like to call them depending on your stack of choice. They’re simply objects that carry data and no behavior. 

To be able to use this pattern in [my REST API](https://github.com/hakant/HackathonPlannerAPI), I’ve created a simple module in TypeScript that allows me to execute commands and optionally get results from them. It’s here: [TypeScriptCommandPattern](https://github.com/hakant/TypeScriptCommandPattern).

In the example below, you can see how a test request is sent down to the executor and a result is returned back.

<br/>
<script src="https://gist.github.com/hakant/837f08708feed35b2d02b99269f0e916.js"></script>

And here is how a “Hello World” style command handler looks like. Notice that it has a clean request and response definitions. At the end of the implementation we make sure that the handler is registered and mapped to the request. In this structure handlers are singleton objects and they can later be resolved by the type of the request and then get executed.

<br/>
<script src="https://gist.github.com/hakant/c4c29c9d6e98fb3a85c674a4b0ad6b28.js"></script>

## Using this pattern in a real world Node.js REST API

One great architectural benefit of this pattern is that it allows you to nicely separate an application into many independent scenarios — [vertical slices](https://lostechies.com/jimmybogard/2015/07/02/ndc-talk-on-solid-in-slices-not-layers-video-online/). Take a look at [these scenarios from Hackathon Planner](https://github.com/hakant/HackathonPlannerAPI/tree/master/scenarios):

![Kent Beck's Feedback Chart](/assets/ContinuousIntegration_Part2/Hackathon_Planner_Scenarios.png)

Each of these TypeScript modules are a vertical slice. They’re the full story. Yes they make use of other external modules when necessary, but when developers read through these scenarios, they get the full picture. And if code sharing between these scenarios are done wisely (with correct abstractions), then a change in one scenario is contained and does not necessarily affect any others.

Let’s take a look at the implementation of one of these scenarios and extract some key properties. Let’s pick the scenario in “GetIdeas.ts”:

<br/>
<script src="https://gist.github.com/hakant/6c38b895807a4d721c8c367a5117826f.js"></script>

* Line 3–4: Imports the base async command handling structure and also the container to register itself at the end of the file.

* Line 6–9: External modules that this handler uses.

* Line 11–12: A module that is used by multiple scenarios. Be careful, I say “used” not “reused”. Remember the “[fallacy of reuse](http://udidahan.com/2009/06/07/the-fallacy-of-reuse/)”. This module (IdeaPrePostProcessor) is used to massage and sanitize the Idea entities before they’re sent to the user interface and before they’re written to the database. Multiple scenarios use this module the exact same way and for the same need. There are no slight variations per scenario, that’s why it’s a right abstraction and that’s why it’s “use”, not “reuse”.

* Line 14–16: Since our handler is an async one (because it’s accessing the database), it implements the AsyncCommandHandler<TRequest, TResponse> and implements the HandleAsync method that returns a Promise of GetIdeasReponse object.

* Line 17–38: Complete vertical implementation of the handler. It’s basically scanning the database for all the ideas, sorting them based on an algorithm, encapsulating them in the response object and return. Notice that at line 25, the “await” keyword is used. It’s a simple and very readable way of representing asynchronous code. It exists in recent versions of TypeScript as well as in ES7.

* Line 42–47: Request and response objects that are used by this handler are defined and exported. These objects should be available to the rest of the application as well.

* Line 50–52: A singleton instance of the handler is instantiated and registered to the type of the request object. Based on application needs, this can be made more complex and sophisticated. Statically typed languages incorporate lots of ideas around IoC containers whereas dynamically typed languages like javascript, not so much. Also keep in mind that these TypeScript generic constructs only exist at design time. They don’t exist in the transpiled javascript and that makes it impossible to discover or scan these constructs at runtime.

## Simple & Stupid ExpressJS Routes

If you have experience with building MVC type web applications or REST API’s you’re probably familiar with the idea of controllers. Routes are the same concept in [ExpressJS](http://expressjs.com). Keeping your routes and controllers nice and clean is a good discipline to have. Remember, we want to contain our core application logic and try not to leak it outside as much as possible. Not to frameworks, not to external libraries.

Hackathon Planner’s most significant route is the Ideas route where ideas are being CRUD and a few further actions are taken against them. Code below shows the routes defined in there. Notice how clean it reads and how all business logic is kept out of it. The only thing these routes do is to pass requests and responses up and down the stream.

<br/>
<script src="https://gist.github.com/hakant/85fbfa3f3d77f82c2b17457a96356d09.js"></script>


## Testing all the Scenarios

The structure of the software created so far lends itself nicely to the subcutaneous testing style discussed earlier. So I take advantage of this by taking the following actions in the tests:

* Remove the ExpressJS layer that normally sits on top. This is a delivery mechanism and I don’t necessarily need it to test my business logic — since I’ve already separated that logic clearly out of this layer.

* Load all scenario handlers together with the real data store (or a realistic emulator).

* Shoot requests & verify responses.

* In some cases, if verifying the response alone isn't sufficient, go further and directly check the data store for verifying the side effects. In my case it wasn’t necessary, I could both act and verify using my handlers alone.

Let’s take a look at a few of these tests. Below is a part of [jasmine](https://jasmine.github.io/) spec that tests inserting and fetching ideas by running several scenarios.

<br/>
<script src="https://gist.github.com/hakant/a2d1c53fdaa5e7c0d99eeb8e8b4022e2.js"></script>

Here are a few key properties of these tests:

* Lines 17–26: Before each test new NoSql tables are created and after each test they’re dropped. This makes every test run in a sandbox isolated from one another. So they can all run in any random order. In general, this is a characteristic of a unit test, not an integration test. So here we have the best of both worlds!

* All tests run against a DynamoDb local emulator. So the feedback coming from the database is real. I trust that the Amazon team behind DynamoDb makes sure that the emulator behaves exactly the same as the real one in the cloud.

* Integration tests are slow right? No, not if you don’t test against user interfaces, or have many number of network calls, or use slow data storage devices. These tests don’t get into any complexity of testing against a UI; they don’t bring up a web server and make excessive number of network calls; and they make use of an in memory database emulator running on the same box. These 15 tests run in ~2 seconds even though each of them setup and tear down their data tables. So you can run quite a lot of these in a few minutes.

* By their nature, integration tests are broad. They give broad feedback. But this is a trade off because when they fail, you have to dig a little deeper to understand exactly what failed in comparison to the unit tests that are tiny and focused. But that’s a tradeoff I like to make in general.

* Having said that, I’m by no means trying to trash the value of unit tests here. If I see the need — like any piece of code that is significant for the system and does interesting things — I’ll go and unit test that part in isolation. Especially code that’s heavy on algorithmic work. So unit tests are valuable when they’re implemented for the right code against the right abstraction.

* Last but not least, these type of tests (i.e subcutaneous) interacts with the system right at the outside of the significant boundary. This means as long as feature requirements don’t change, these tests remain valid and useful. They can survive large refactorings because they’re not coupled to the implementation details! This is a huge deal in my opinion.

In next and the last part of this series, I would like to write a few things about my experience with setting up a CD pipeline using [CircleCI](https://circleci.com), which transpiles all TypeScript files, installs dependencies, runs tests and deploys to [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/).

Thanks for reading.

~ Hakan