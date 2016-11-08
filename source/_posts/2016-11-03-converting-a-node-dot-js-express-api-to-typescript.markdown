---
layout: post
title: "5 Quirks of Converting a Node.js Express API to TypeScript"
date: 2016-11-08 21:32:05 +0100
comments: true
categories: [typescript,node.js,express]
footer: true
sharing: true
description: TypeScript language and the tooling around it can help all sizeable javascript projects to become more maintainable.
image:  
---

I started using TypeScript as part of a project I'm working on. If you're new to TypeScript or 
aren't yet sure about the benefits it can bring to your Javascript codebases, I recommend seeing 
[this presentation from Anders Hejlsberg (Build 2016)](https://channel9.msdn.com/Events/Build/2016/B881).

One thing I really like about TypeScript is that it's a superset of Javascript, which means all Javascript 
you wrote until now is already valid TypeScript. This makes it possible for anyone to come 
into TypeScript and adopt it as much or as little as they want. Anders calls this "turning the knob" 
experience. If you want to go all the way in with the type system, you can do so. Or if you only want to 
use it in certain parts of your codebase, you can do that too.

As I learned more about TypeScript, I got convinced that I can benefit from it in most of my Javascript projects, 
in particular, the ones that involve rich business domains. As an experiment, I started 
converting [the backend of Hackathon Planner](https://github.com/hakant/HackathonPlannerAPI), written in [Node.js](https://nodejs.org/en/) 
and [Express](http://expressjs.com/), into TypeScript. 

All in all, it's been a smooth experience but if 
you're a developer yourself, you know that this type of refactorings never happen on a straight line, there 
are always bumps and quirks along the way.   

Here are a number of things I experienced as I gradually converted all of the codebase from ES6 Javascript 
to TypeScript. This repository is open source and on [GitHub](https://github.com/hakant/HackathonPlannerAPI). 
TypeScript version of the code might still be on a separate branch if I haven't merged it yet.

## 1. At transition, module loading can get tricky in Node.js

TypeScript shares the [same concept of a module as ES6](https://www.typescriptlang.org/docs/handbook/modules.html). 
TypeScript's module import and export syntax is very similar to its ES6 counterpart but the [CommonJS 
module system](https://nodejs.org/docs/latest/api/modules.html) that Node.js uses is different.

I wanted to slowly port my Javascript to TypeScript so that I can continue making releases in between. 
Obviously, this means that part of the code would be transpiled from TypeScript while the other part is 
originally Javascript. In many occasions I had to load a module from a .js file and the target
module was transpiled from a .ts (TypeScript) file. [This can cause issues](https://github.com/Microsoft/TypeScript/issues/2719).

Look at the code below and see what happens when "ideas.js" has to load a module from "AdminRepository.ts". 
All of a sudden you have to use the "default" property to be able to access the exported object. 
Eventually, when the consumer (idea.js) is also converted to TypeScript, the problem will go away.
<br/>

<script src="https://gist.github.com/hakant/d68576a8d9cc9ed85e8d3c649ad8a856.js"></script>

## 2. Adding Type Definition Files to a Project

Type definition files let you consume third party libraries as if they were written in TypeScript. So 
you get all the benefits of type checking, intellisense and all other TypeScript design & compile time
goodness even when the third party library is written in pure Javascript.

Before 2.0 release of TypeScript, finding and using these type definition files could get a bit [painful and 
confusing](https://www.davevoyles.com/2016/02/16/how-to-add-type-definitions-to-a-typescript-project/) - 
same goes for [creating them](https://medium.com/@mweststrate/how-to-create-strongly-typed-npm-modules-1e1bda23a7f4#.7ur75n1hf).
TypeScript 2.0 fixed some of these issues by introducing a [simplified declaration file
acquisition model](https://blogs.msdn.microsoft.com/typescript/2016/09/22/announcing-typescript-2-0/#simplified-declaration-file-dts-acquisition).

In node.js, in order to grab the underscore.js package, you'd use:
>npm install underscore --save

Now, if you also want to use this package from TypeScript, with strong typing, you'll have to download
the necessary type declaration file as well:
>npm install @types/underscore --save

Starting with TypeScript 2.0, these type definition files are regular npm packages.

After downloading the type definition file, VS Code stops complaining. Furthermore, I get all type safety 
features and intellisense while using this library, which is what I want and why I use TypeScript. 

![Types Packages](/assets/Node_Express_TypeScript/types_packages.png)

On the other hand, sometimes a library you're using might not have a type definition file. For example, I'm using 
a passport extension library called "passport-github2" which doesn't have a definition file yet 
(and probably never will). Inside any TypeScript file that uses this library, VSCode complains that it doesn't 
understand this library. TypeScript compilation also results in the same error. But even though the typing file 
is missing, the actual library is there and it works during run-time.

But of course we've a problem here. The problem is that the TypeScript compiler will complain about this missing type definition file
forever. Being a good developer, what you want to do is to avoid having these kinds of error messages lying around
and being ignored.

![Missing Typings](/assets/Node_Express_TypeScript/missing_typing.png)

Having said this, it's worth mentioning that 
[people are complaining](https://medium.com/@sam.verschueren/the-one-thing-that-keeps-me-from-using-typescript-839e9bd464d7#.fg5tvva5g) 
about this aggressive behavior of TypeScript, as it eagerly wants all 3rd party packages to have type definition files. But
there's probably [a solution coming](https://github.com/Microsoft/TypeScript/pull/11446).

Until then, there are 2 possible solutions: 

* Set "allowJs" compiler option to true (in ts.config file). Although I should warn you that this will not only allow 
third party packages to be consumed without type definitions, it will also allow any developer to put pure 
javascript code into your project. 
* Another solution is to declare an [ambient module](https://www.typescriptlang.org/docs/handbook/modules.html#ambient-modules) 
for that library in a separate file - either marking it with 'any' keyword or with proper types.

I chose the latter, because I noticed that "allowJs" flag has far bigger consequences. It silences a bunch of 
other types of errors/warnings which I would like to be informed about. Here is how those ambient modules 
look like:

![Ambient Modules](/assets/Node_Express_TypeScript/ambient_modules.png)

## 3. Incorrect / Incomplete Type Definition Files

Sometimes a type definition file (.d.ts file) turns out to be incorrect or incomplete. The screenshot
below shows a situation where the package "aws-sdk" has an incomplete type definition. It doesn't 
know that "endpoint" property exists even though it does exist in [AWS documentation](http://docs.aws.amazon.com/amazondynamodb/latest/gettingstartedguide/GettingStarted.NodeJs.01.html).

![Incorrect type declarations](/assets/Node_Express_TypeScript/incorrect_d.ts_file.png)

I literally went into the type declaration file I downloaded via npm and added the field "endpoint"
in there to fix the problem. Probably I have to commit this file into source control now. Unfortunately 
I couldn't find a best practice that explains how to properly handle these situations. For now, I'll go
with the solution of manually fixing the definition file and adding it into the source control. Below you
can see the patch I made to the declaration file of AWS-SDK (line commented out).

![ClientConfigPartial-AWS-SDK](/assets/Node_Express_TypeScript/clientconfigpartial_d.ts.png)


## 4. Dynamic Augmentation of Javascript Objects

A very common pattern in javascript is to dynamically extend or augment an object with new or complementary
behavior. Here below, I'm using [bluebird.js](http://bluebirdjs.com) to convert a callback oriented API
to a Promise based API. All of a sudden though, TypeScript doesn't know about these new "async" methods that 
bluebird has plugged into the dynamodb client. 

![Bluebird-Promisify-All](/assets/Node_Express_TypeScript/promisify_all.png)

The solution is to either declare this object as a new interface or mark it as "any", 
as suggested [here](https://github.com/Microsoft/TypeScript/issues/8685)). I've chosen the first, a more type-safe
approach, and declared my new type in the "aws-sdk" type declaration file, which solved the problem in 
a nicer way imho. See the DynamoDBAsyncClient interface definition below:

![DynamoDB_Async_Client_Declaration](/assets/Node_Express_TypeScript/dynamodb_async_client_declaration.png)

## 5. Using import or export in your TypeScript file turns it into a module

Both in TypeScript and ES6, any file containing a top-level import or export is considered a module. So 
right after you use one of these keywords in your Javascript or TypeScript file, it will start behaving like 
a module. I stumbled upon this while I was trying to implement an interface that I had created in a separate 
file. 

Let's first look at a simple case that doesn't make use of modules. Here below there's an interface and its 
implementation right below it. See how AdminRepository.ts refers to IAdminRepository interface without 
any ceremony.
<br/>

<script src="https://gist.github.com/hakant/ce9a6521a16d871bb652c2f41299312d.js"></script>

<script src="https://gist.github.com/hakant/7c25b55660deffb5556df2357ab5708b.js"></script>

Now look at these other two files, in which the interface exposes a third party object and therefore has to 
import that third party module. Note that index.ts can't just refer to that interface anymore, without 
actually importing it.
<br/>

<script src="https://gist.github.com/hakant/126bbcdaa477680e4ace2021f9dedd3e.js"></script>

<script src="https://gist.github.com/hakant/88d0907ab7f3c090b4d4404a31c079c5.js"></script>

By the way, I also noticed that [TypeScript coding guidelines](https://github.com/Microsoft/TypeScript/wiki/Coding-guidelines)
do not recommend using "I" as a prefix for interface names. So I need to hold my C# reflexes. 

There are more than a few things that one has to do in order to make the TypeScript compiler happy. For anyone
with a "Javascript mindset" this may feel like unnecessary burden. But once you make this investment, 
the amount of help and insight you get from the TypeScript compiler and tooling around it is amazing.

Thanks for reading.

~ Hakan
