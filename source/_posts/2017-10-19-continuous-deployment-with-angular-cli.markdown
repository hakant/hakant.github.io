---
layout: post
title: "Continuous Deployment with Angular CLI, Docker, Travis CI and Azure"
date: 2017-10-19 08:39:36 +0100
comments: true
categories: ContinuousDelivery,Azure,Angular,Docker,TravisCI
footer: true
sharing: true
description: In this post, I'm setting up a continuous delivery pipeline using Angular CLI, Docker Containers, Travis CI and Azure
image: http://www.hakantuncer.com/assets/CD_Docker_Angular_Azure/dockerangular.png
---

These days when I start writing code for a new project, I feel a sense of urgency to create an end to end development workflow as soon as possible. I love the phrase “[walking skeleton](http://alistair.cockburn.us/Walking+skeleton)”. So think of this end to end workflow as a walking skeleton of a [continuous delivery pipeline](https://en.wikipedia.org/wiki/Continuous_delivery). It’s basically taking a smallest possible version of your application and setting up the necessary infrastructure to be able to continuously build, run analysis, test and deploy to a production-like environment.

Please don’t confuse this idea with a waterfall concept. Your build, test and deployment scripts don’t have to be future proof. They don’t have to cover all the possible scenarios that you might need in the future. Far from it. The idea is to have (just) enough capability in the pipeline to be able to cover a (vertical) slice of your application. It should have enough “meat” to get you going.

Also note that the definition of “vertical” has grown larger with the [DevOps](https://en.wikipedia.org/wiki/DevOps) way of doing things. You should probably not stop at automated deployments either. Can you also monitor this first small version of your application already — both in terms of usage and errors ? Do you already have a central place to track bugs, features and collaborate with other stakeholders? So yes, the idea is to really get your whole cycle up and running as soon as possible. But I digress..

##.travis.yml file

[YAML](https://en.wikipedia.org/wiki/YAML) files have become a standard way of configuring continuous integration & delivery pipelines. [Travis CI is no different](https://docs.travis-ci.com/user/customizing-the-build/).

YAML file below tells Travis to;

* (lines 4-5) Install node 7.7.4
* (lines 7-8) Make docker available.
* (lines 10-20) Install and configure the latest stable version of Chrome
* (line 23) Download all javascript packages that the Angular app uses
* (line 26) Run all Angular component tests and do not watch files, quit immediately after.
* (line 27) Run all end to end protractor tests. Think of this like Selenium but a bit more specialised for Angular.
<br/>

<script src="https://gist.github.com/hakant/35b9259651d2b03b3d73cb92be0aaadb.js"></script>

* (lines 29–30) If the build is successful and branch is master, prepare for packaging and deployment.
* (line 31) Build the angular app using the CLI. This will transpile all .ts files and create .js bundles using [webpack](https://webpack.github.io/). It will eventually create a folder called “dist” which will include all the release artefacts that are ready for deployment.
* (line 32) Build a docker image with the specified name and based on the specified docker file. You can see the [docker file](https://github.com/hakant/Roost-Angular/blob/master/.docker/nginx.dockerfile) and the [nginx configuration](https://github.com/hakant/Roost-Angular/blob/master/.docker/nginx.conf) if you’re interested.
* (line 33) Tag the image that’s just been created using the SHA of the latest git commit. This way any docker file can be reverse engineered to it’s exact source version.
* (lines 34-35) Login to docker hub and push the image (using secret environment variables which I’d set earlier on Travis). You can see a list of images that I’ve already pushed on [docker hub](https://hub.docker.com/r/hakant/roost-angular/tags/).

[At the time of this writing there were 20 builds in the history](https://travis-ci.org/hakant/Roost-Angular/builds).

So last step above pushes the brand new docker image to Docker Hub but how does it get deployed on Azure ?

## Continuous Deployment of Docker Images to Azure

There’s one more step which is kinda invisible and that’s the deployment of the new docker image to an Azure Web App as a container. This is achieved through a [webhook that is setup behind the scenes in Docker Hub and Azure Portal](https://docs.microsoft.com/en-us/azure/app-service/containers/app-service-linux-ci-cd). There are a few commands that need to be executed against the Azure CLI:

Enabling container continuous deployment feature:

```text
az webapp deployment container config -n Roost-Angular -g Roost -e true
```

Obtaining the WebHook URL:

```text
az webapp deployment container show-cd-url -n roost-angular -g roost
```

Pasting the WebHook URL on Docker Hub:

![Docker Hub WebHook](/assets/CD_Docker_Angular_Azure/DockerHub_WebHook.png)

Once this is done, whenever a new image is pushed into Docker Hub, this webhook is automatically invoked. I’ve noticed that if I make a GET call to this URL manually, it also kicks off a new deployment. Straightforward.

And on Azure Portal there’s a designated “Docker Container” tab under the Azure Web App for this purpose. Notice the name of the image and the webhook url (which is concealed by default). One docker convention to keep in mind is that when a :tag is not specified it means :latest.

![Docker Container Tab Azure](/assets/CD_Docker_Angular_Azure/DockerContainer_Tab_Azure.png)

## More information
<br/>

* [Run a custom Docker Hub image in Azure Web App for Containers](https://docs.microsoft.com/en-us/azure/app-service/containers/quickstart-custom-docker-image)
* [Travis CI — Using Docker in Builds](https://docs.travis-ci.com/user/docker/)
* [Push images to Docker Cloud](https://docs.docker.com/docker-cloud/builds/push-images/)
* [Stories describing how to do more with the Angular CLI](https://github.com/angular/angular-cli/wiki/stories)
* [The Ultimate Angular CLI Reference Guide](https://www.sitepoint.com/ultimate-angular-cli-reference/)
* [ng test](https://github.com/angular/angular-cli/wiki/test), [ng e2e](https://github.com/angular/angular-cli/wiki/e2e), [ng build](https://github.com/angular/angular-cli/wiki/build)

Thanks for reading.

~Hakan
