---
layout: post
title: "Continuous Deployment with Node.js, DynamoDB, AWS Elastic Beanstalk and CircleCI"
date: 2017-05-25 07:48:55 +0200
comments: true
categories: [CircleCi,ContinuousDelivery,NodeJS,AWS,DynamoDb]
footer: true
sharing: true
description: In this post, I'm taking a deep dive into setting up a continuous delivery pipeline using CircleCI.
image: https://cdn.scotch.io/1/KetK0AgfTuNR0bVTiVMn_continuous-integration-with-circle-ci.png
---

These days everybody is talking about DevOps and CI pipelines. Unicorn tech companies are building DevOps tools and CI pipelines as a service and bragging about their user friendly design and how easy it is to setup these pipelines on their branded platforms. This is all nice and useful, however, I can’t help but see another [silver bullet syndrome](https://www.youtube.com/watch?v=qamzvLfX-Zo).

Anyone who’s building non-trivial software systems long enough know that software development is hard. Especially with sizeable teams that are larger than only a few devs, it’s very easy to turn the system into [a big ball of mud](https://vimeo.com/171704597) and get the codebase into a state that is difficult to understand and to reliably test in an automated fashion.

If you don’t have reliable automated tests with realistic test cases and good coverage, your CI pipeline is just a lie. So please stop treating your CI pipelines (that probably took you 10 minutes to create these days) as silver bullets.

All that said, if you have actually done the REAL work and payed attention to your codebase, your architecture and treated your tests as highly as your production code, then you can now go ahead and create your CI pipeline in 10 minutes and it will benefit you tremendously.

This is a follow-up on my previous blog posts where I did the real work and made my codebase clean and testable. You can find them here in two parts: [Part 1](/blog/2017/02/17/setting-up-continuous-integration-and-delivery-for-a-nodejs-rest-api/) and [Part 2](/blog/2017/03/25/setting-up-continuous-delivery-for-a-node-dot-js-rest-api-part-2/). Having said that, these are no pre-requisites to this post by any means. Just continue reading if you’re only interested in the Continuous Delivery Pipeline itself, you should be perfectly fine.

##Why CircleCI

I know it won't sound very exciting but there’s actually no particular reason. My two biggest requirements were FREE and SaaS. I obviously don’t want to pay a monthly fee for the CI pipeline of an open source hobby project. And I just want to use it SaaS style and don’t want to own anything. To name a few alternatives to [CircleCI](https://circleci.com/) which are free for non-commercial open source projects and SaaS; there is [VSTS](https://www.visualstudio.com/team-services/), [AppVeyor](https://www.appveyor.com) and [Travis CI](https://travis-ci.org). I used the first two, they’re all very good offerings, I would also like to carve some time to try Travis in the future if I can.

##Configuring the Pipeline with CircleCI

There’s an [in depth documentation](https://circleci.com/docs/1.0/configuration/) that describes the ins and outs of CircleCI and how to configure your pipeline for various needs and scenarios. It starts with:

>CircleCI automatically infers settings from your code, so it’s possible you won’t need to add any custom configuration.
><br/><br/>
>If you do need to tweak settings, you can create a circle.yml in your project’s root directory. If this file exists, CircleCI will read it each time it runs a build.

The automatic inference of settings does not completely work for [my codebase](https://github.com/hakant/HackathonPlannerAPI). There are certain build steps that I want to override and specify my specific need — like transpiling TypeScript and running tests against a DynamoDB emulator, so I need to create a __circle.yml__ file in the project root directory.

##Primary Steps of a CircleCI Pipeline

When we talk about a CI pipeline, we’re actually talking about a series of steps that are executed one after the other in a deterministic and repeatable manner.

>The __circle.yml__ file has seven primary sections. Each section represents a phase of the Build-Test-Deploy process:
><br/><br/>
>__machine__: adjust the behavior of the virtual machine (VM)
><br/><br/>
>__checkout__: checkout and clone code from a repository
><br/><br/>
>__dependencies__: install your project’s language-specific dependencies
><br/><br/>
>__database__: prepare a database for tests
><br/><br/>
>__compile__: compile your project
><br/><br/>
>__test__: run your tests
><br/><br/>
>__deployment__: deploy your code to your web servers

For each of these sections CircleCI automatically infers commands based on the characteristics of the codebase.

>You can specify when to run custom commands relative to CircleCI’s inferred commands using three special keys:
><br/><br/>
>__pre__: run before inferred commands
><br/><br/>
>__override__: run instead of inferred commands
><br/><br/>
>__post__: run after inferred commands

##CircleCI configuration of Hackathon Planner

I want to create the CI pipeline for [Hackathon Planner’s REST API](https://github.com/hakant/HackathonPlannerAPI). This API is built with [Node.js](https://nodejs.org), [ExpressJS](http://expressjs.com), [TypeScript](http://www.typescriptlang.org), [DynamoDB](https://aws.amazon.com/dynamodb/) and deployed to [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/).
<br/>

###machine

My REST API is running on node 5.11. So I want to use the same version of node to run my tests. 

I also need Java Development Kit (jdk) to run the DynamoDB emulator. My Unit/Integration tests will run against this emulated DynamoDB instance.

After node and jdk is installed on the machine, I need to run 3 commands to download, extract and run DynamoDB emulator (under section “post”):

```text

machine:
  node:
    version: 5.11
  java:
    version: openjdk7
  post:
    - curl -k -L -o dynamodb-local.tgz http://dynamodb-local.s3-website-us-west-2.amazonaws.com/dynamodb_local_latest.tar.gz
    - tar -xzf dynamodb-local.tgz
    - "java -Xms1024m -Xmx1024m -Djava.library.path=~/DynamoDBLocal_lib -jar ~/DynamoDBLocal.jar --port 8000": background: true

```

<br/>
###compile

In this part of the build step, I’m taking over the inferred behavior of CircleCI using the “override” keyword.

```text
compile:
  override:
    - tsc -p .
```

“tsc” is the TypeScript compiler and the “-p” flag tells it to compile the project given a valid configuration file (tsconfig.json). In my codebase, this file is located in the root directory.

“tsconfig.json” file below tells the compiler to transpile all TypeScript files in the folder (recursively) except the contents of the “node_modules” folder. For more info on TypeScript compiler options, [checkout this documentation](https://www.typescriptlang.org/docs/handbook/compiler-options.html).

```json
{
    "compilerOptions": {
        "module": "commonjs",
        "sourceMap": true,
        "target": "es6",
        "moduleResolution": "node",
        "allowJs": false
    },
    "exclude": [
        "node_modules"
    ]
}
```

<br/>
###test

Now that Node.js and DynamoDB is installed on the build machine (actually a container) and the application is transpiled into ES6 JavaScript that node can directly understand, it’s now time to run all the tests.

Once again, I’m taking over the inferred behavior of CircleCI using the “override” keyword and call into a predefined npm script.

```text
test:
  override:
    - npm test
```

When the “npm test” command runs, behind the scenes npm looks into the “scripts” section of the “package.json” file to figure out what to execute.

```json
// package.json

"scripts": {
    "start": "node ./bin/www",
    "test": "node ./tests/runUnitTests.js"
  }
```  

Via this lookup, it’s going to find and execute the “runUnitTests.js” file, which loads the jasmine test framework and executes the tests via another configuration file — “jasmine.json”.

```javascript
// runUnitTests.js

var Jasmine = require('jasmine');
var jas = new Jasmine();

jas.loadConfigFile('./tests/unit/jasmine.json');
jas.execute();
```

“jasmine.json” file specifies where the tests and test helpers are and adjusts certain test behaviour. For example here I specified that I don’t want tests to stop running on failures because I would like to get a complete feedback. I also specified that tests should run in random order. All my tests in this project (unit / integration) are isolated from each other, so I should be able to run them in any order.

```text
// jasmine.json

{
  "spec_dir": "tests/unit",
  "spec_files": [
    "./*[sS]pec.js"
  ],
  "helpers": [
    "./helpers/*.js"
  ],
  "stopSpecOnExpectationFailure": false,
  "random": true
}
```

Only when all tests run successfully, will the build continue to the next step and deploy the application to production.


##From CI to CD — Taking the next step

At this point I’ve created a complete CI pipeline where at each commit to a branch a build will kick in, compile the codebase and run all the tests. Now all developers can integrate their work as early as possible and get continuous feedback from the build server.

However, if you and your team has invested into the CI pipeline enough that you trust the depth and breadth of feedback you receive from it, you can take the next step into continuous deployment.

Of course nothing forces you to deploy directly to production, automatically deploying to a staging environment for a final round of Q/A is definitely a more common approach.

<br/>
###dependencies & deployment

Below are the build steps where I install [AWS Elastic Beanstalk CLI](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3.html) with its dependencies and call the deploy command for the right branch and profile.

```text
dependencies:
  pre:
    - sudo apt-get install python-dev
    - sudo pip install 'awsebcli==3.7.4' --force-reinstall

deployment:
  production:
    branch: master
    commands:
      - eb deploy --profile default

```

It turns out that “python-dev” tools is a pre-requisite for AWS EB CLI. Notice that I only deploy the master branch to production. None of the other branch builds will run this deployment step.

If you think about what needs to happen behind the scenes for an actual deployment process, this configuration seems to be too little. For example, how does EB CLI know where to deploy the application in AWS ? or how does CircleCI authenticate with AWS on my behalf to make this deployment? So indeed, this is not the full story. There are two more pieces of the puzzle that I want to show you quickly.

By convention, “.elasticbeanstalk/config.yml” is the path that EB CLI looks for during the deployment. One of the key things in this file is the first section where the environment name is defined. “hackathonplanner” is the environment that is uniquely defined under my AWS account and this is how EB CLI knows where to deploy the package to.

```text
//.elasticbeanstalk/config.yml

branch-defaults:
  master:
    environment: hackathonplanner
global:
  application_name: Hackathon-Planner
  default_ec2_keyname: null
  default_platform: 64bit Amazon Linux 2016.03 v2.1.3 running Node.js
  default_region: eu-central-1
  profile: eb-cli
  sc: git
```

The answer of the second question, which is, “how does CircleCI authenticate on my behalf to make this deployment happen” is below:

![CircleCI AWS Keys Page](/assets/ContinuousIntegration_Part3/AWS-Keys.png)

As you see, CircleCI has built in first class support for AWS. Just like it’s recommended on the configuration page above, I’ve created a unique [IAM user](https://aws.amazon.com/iam/) in AWS and gave it deployment permissions to the “hackathonplanner” environment. Then I copy and pasted the “Access Key ID” and “Secret Access Key” over from AWS to CircleCI. Now CircleCI can deploy to my environment on my behalf. It’s that easy.

##Build results

At the time of this writing I’ve got [72 builds queued and ran](https://circleci.com/gh/hakant/HackathonPlannerAPI) both green and red.

If you look at one of the successful builds, let’s take [the last one](https://circleci.com/gh/hakant/HackathonPlannerAPI/72), you can see each and every step that were executed and their respective logs. 

After uncollapsing the test step and reading through the logs, I noticed that for some reason my tests can not tear down DynamoDB tables anymore. It doesn’t fail any test since each test creates new tables with random names, so the build is still green and everything is fine. I need to dig in and understand what has changed in the meantime — while I was not paying attention. 

This reminded me of [a recent talk from Ian Cooper](https://vimeo.com/channels/1203692/204428794) where 17 mins in, he asks the audience if they deployed a piece of software and left it alone for a year, what would they expect when they come back. Would it still be up and running correctly or not? So the point is, things keep changing around our software (especially in the cloud) and when nothing is done, our software decays by itself.

Anyway, after this little segway into a different subject, I think it’s time to stop here. Thanks for reading and if you have any comments or questions you can write it below or tweet me at [@hakant](https://twitter.com/hakant)!