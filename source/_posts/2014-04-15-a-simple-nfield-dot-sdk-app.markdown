---
layout: post
title: "A simple Nfield.SDK app"
date: 2014-04-15 20:14:15 +0200
comments: true
categories: [nuget, nfield]
footer: true
sharing: true
---

In a previous blog post I covered the step by step process of [publishing Nfield.SDK to NuGet]({{ root_url }}/blog/2014/04/13/publishing-nfield-dot-sdk-to-nuget/). 

After publishing Nfield.SDK, a natural follow up is to create a simple app that installs Nfield.SDK through NuGet and does something useful with it.

>To learn more about Nfield or to <a href="http://nfieldmr.com/get-started.aspx" target="_blank">get started</a> visit <a href="http://nfieldmr.com/" target="_blank">nfieldmr.com</a>

## Installing Nfield.SDK in a Console Application

What is better than a windows console application to prove that it's really easy to install and use Nfield.SDK. Let's fire up Visual Studio, create a Console Application and install Nfield.SDK through NuGet.

Opening the package manager for my console app and typing nfield in the search box brings me the Nfield.SDK NuGet package:

![Nfield SDK Online in Nuget](/assets/A_Simple_NfieldSDK_App/NfieldSDK_Online_InNuget.png)

After clicking install, I see that Nfield.SDK is automatically installed and referenced from my project.

![Nfield SDK Installed In My Project](/assets/A_Simple_NfieldSDK_App/NfieldSDK_In_My_Project.png)

Wait a moment... I only installed Nfield.SDK but now I see that "Json.NET", "Microsoft ASP.NET Web API Client Libraries" and "Microsoft .NET Framework 4 HTTP Client Libraries" also got installed. As discussed in the [previos blog post]({{ root_url }}/blog/2014/04/13/publishing-nfield-dot-sdk-to-nuget/) NuGet is smart enough to recursively install packages and all of their dependencies.

In this case Nfield.SDK has a direct dependency to "Microsoft ASP.NET Web API Client":

![Nfield SDK Dependencies on NuGet](/assets/A_Simple_NfieldSDK_App/Nfield.SDK_Dependencies.png)

And "Microsoft ASP.NET Web API Client" has direct dependency to the rest:

![WebApi Client Dependencies on NuGet](/assets/A_Simple_NfieldSDK_App/WebApiClient_Dependencies.png)

The point here is that NuGet took care of all the dependencies of my application in one click of an install button.

## Getting ready for coding

Focusing back to the console application, I'm going to create an app that is going to add an interviewer to my domain on Nfield. In order to do that, I need one more ingredient to take care of: an [Inversion of Control Container](http://en.wikipedia.org/wiki/Inversion_of_control).

Nfield.SDK implementation is a little opinionated about the usage of an IoC Container together with it. It won't dictate you which one to use that's why it has an initializer that accepts some key registration methods:

``` csharp
		/// <summary>
        /// Method that registers all known types by calling the delegates provided.
        /// This method must be called before using the SDK.
        /// </summary>
        /// <param name="registerTransient">Method that registers a Transient type.</param>
        /// <param name="registerSingleton">Method that registers a Singleton.</param>
        /// <param name="registerInstance">Method that registers an instance.</param>
        public static void Initialize(Action<Type, Type> registerTransient, 
                                      Action<Type, Type> registerSingleton,
                                      Action<Type, Object> registerInstance)
```
<br>
>If you don't know what an IoC Container is there is nothing to worry about. If you just copy the short setup code that I'm going to share in a second you're ready to use Nfield.SDK. You don't have to use IoC for your application logic. Again, if you're new to IoC Containers I recommend reading [this paper from Martin Fowler](http://martinfowler.com/articles/injection.html). 

An IoC container that we frequently use at NIPO Software is [Ninject](http://www.ninject.org/). I'll install it to my project using NuGet again (only the top one in the list below is sufficient).

![Adding Ninject To My App](/assets/A_Simple_NfieldSDK_App/AddingNinjectToMyApp.png)

## Nfield.SDK in action

Good. Now I'm ready to write my actual business logic. Signing into my Nfield domain and creating an interviewer. Here is how the code looks like:

``` csharp
using System;
using Nfield.Infrastructure;
using Nfield.Models;
using Nfield.Services;
using Ninject;

namespace TestNfieldApp
{
    class Program
    {
        static void Main(string[] args)
        {
            using (IKernel kernel = new StandardKernel())
            {
                InitializeNfield(kernel);

                const string serverUrl = "https://api.nfieldbeta.com/v1";

                // First step is to get an INfieldConnection which provides services used for data access and manipulation. 
                var connection = NfieldConnectionFactory.Create(new Uri(serverUrl));

                // User must sign in to the Nfield server with the appropriate credentials prior to using any of the services.
                connection.SignInAsync("H", "DA", "******").Wait();

                var interviewersService = connection.GetService<INfieldInterviewersService>();

                interviewersService.AddAsync(new Interviewer
                {
                    ClientInterviewerId = "hakantun",
                    UserName = "hakant",
                    FirstName = "Hakan",
                    LastName = "Tuncer",
                    EmailAddress = "h.tuncer@niposoftware.com",
                    Password = "hakan12"
                }).Wait();
            }
        }

        /// <summary>
        /// Example of initializing the SDK with Ninject as the IoC container.
        /// </summary>
        private static void InitializeNfield(IKernel kernel)
        {
            DependencyResolver.Register(type => kernel.Get(type), type => kernel.GetAll(type));

            NfieldSdkInitializer.Initialize((bind, resolve) => kernel.Bind(bind).To(resolve).InTransientScope(),
                                            (bind, resolve) => kernel.Bind(bind).To(resolve).InSingletonScope(),
                                            (bind, resolve) => kernel.Bind(bind).ToConstant(resolve));
        }
    }
}
```

After I run this application and go to the interviewers page of my domain, I see that the inverviewer I just added is now listed there. Below is a screenshot from the beta version of [Nfield Management Interface](https://manager.nfieldbeta.com/) (authentication required).

![Interviewer Added Successfuly](/assets/A_Simple_NfieldSDK_App/InterviewerAdded_Success.png)

The code I used gets a reference to INfieldInterviewersService and adds an interviewer using this interface.

``` csharp
var interviewersService = connection.GetService<INfieldInterviewersService>();

interviewersService.AddAsync(new Interviewer
{
	ClientInterviewerId = "hakantun",
	UserName = "hakant",
	FirstName = "Hakan",
	LastName = "Tuncer",
	EmailAddress = "h.tuncer@niposoftware.com",
	Password = "hakan12"
}).Wait();
```

Nfield.SDK has many service interfaces that allow clients to control various aspects of their research projects.

``` csharp
public static void Initialize(Action<Type, Type> registerTransient, 
                                      Action<Type, Type> registerSingleton,
                                      Action<Type, Object> registerInstance)
{
    registerTransient(typeof(NfieldConnection), typeof(NfieldConnection));
    registerTransient(typeof(INfieldInterviewersService), typeof(NfieldInterviewersService));
    registerTransient(typeof(INfieldSurveysService), typeof(NfieldSurveysService));
    registerTransient(typeof(INfieldSurveyDataService), typeof(NfieldSurveyDataService));
    registerTransient(typeof(INfieldBackgroundTasksService), typeof(NfieldBackgroundTasksService));
    registerTransient(typeof(INfieldSurveyScriptService), typeof(NfieldSurveyScriptService));
	registerTransient(typeof(INfieldFieldworkOfficesService), typeof(NfieldFieldworkOfficesService));
	registerTransient(typeof(INfieldMediaFilesService), typeof(NfieldMediaFilesService));
	registerTransient(typeof(INfieldLanguagesService), typeof(NfieldLanguagesService));
	registerTransient(typeof(INfieldTranslationsService), typeof(NfieldTranslationsService));
	registerTransient(typeof(INfieldSurveySettingsService), typeof(NfieldSurveySettingsService));
	registerTransient(typeof(INfieldSurveyResponseCodesService), typeof(NfieldSurveyResponseCodesService));
	registerTransient(typeof(INfieldAddressesService), typeof(NfieldAddressesService));
	registerTransient(typeof(INfieldHttpClient), typeof(NfieldHttpClient));
	registerTransient(typeof(IFileSystem), typeof(FileSystem));
}
```

I think that's a post. I hope I could shed some light on getting up and running with Nfield.SDK and the various things that you can do with it.

Thanks for reading.

