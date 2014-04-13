---
layout: post
title: "Publishing Nfield.SDK to NuGet"
date: 2014-04-13 18:34:32 +0200
comments: true
categories: [nuget, nfield, .net, microsoft]
footer: true
sharing: true
---

A couple of days ago I published Nfield.SDK to NuGet. In this blog post I want to describe the steps that are involved. Obviously this process will equally apply to publishing / consuming any other software library or package using NuGet.

## What is Nfield SDK

Me and my colleagues at [NIPO Software](http://www.niposoftware.com/) are building the next generation data collection platform called [Nfield](http://www.nfieldmr.com/). Nfield has an [open source SDK](https://github.com/NIPOSoftware/Nfield-SDK) that market research offices can use to streamline and automate setting up their research projects. 


## NuGet and Package Management Systems
If you're a .NET developer or familiar with Microsoft development environment you probably know what Nuget is. Nuget is an open source package manager for Microsoft development platform (including .NET). Package management systems make poor developers' lives rock (well almost). It eases everything about installing libraries and managing dependencies. It also opens up lots of automation possibilities - like integrating dependency management into the build process. The home of nuget is [www.nuget.org](http://www.nuget.org).

Most platforms have their own package management systems. Ruby has [RubyGems](https://rubygems.org), Node.js has [npm](https://www.npmjs.org), Apache has [Maven](http://maven.apache.org) and it works with multiple platforms but it's primarily used by the Java community. Package managers don't only exist in the context of software development. There are also system-wide package managers to install components and applications for operating systems. For more information check out [Package Management System](http://en.wikipedia.org/wiki/Package_management_system) and [List of Package Management Systems](http://en.wikipedia.org/wiki/List_of_software_package_management_systems) on Wikipedia. 

##Step by step - Publishing Nfield.SDK to NuGet
<br><br>
### 1. Cloning Nfield.SDK from GitHub
When it comes to Git I have a couple of ways to interact. I either directly use the Git commands from a shell or [GitHub for Windows](https://windows.github.com/) and lately I'm also very happy with the Git integration of Visual Studio 2013. In this case I'll just go ahead and use GitHub for Windows to clone Nfield.SDK to my local repository.   

![Cloning Nfield SDK](/assets/NfieldSDK_Nuget/NfieldSDK_GitHub_For_Windows.png)

After cloning is finished, now all files are on my local repository.

![Nfield SDK Folder](/assets/NfieldSDK_Nuget/NfieldSDK_Folder.png)  
<br>

### 2. Creating a .nuspec file for Nfield.SDK project

.nuspec files are XML manifests which describe nuget packages. A .nuspec manifest is used to build a nuget package and it's also stored inside the package after the package is created. There is a [.nuspec reference](http://docs.nuget.org/docs/reference/nuspec-reference) in nuget.org.

So whenever we want to create a nuget package, we first need to create a .nuspec file. Nuget.exe command line utility allows us to create .nuspec files from .csproj files. You can also create a blank .nuspec file but pointing to an existing .csproj file makes things a lot easier since it's automatically detecting some of the metadata for us. 

``` bash run this command on the directory where the .csproj file is located
nuget spec
```

After running this nuget command NField.SDK.nuspec file is created for us under the same directory.

![Nfield SDK nuspec file](/assets/NfieldSDK_Nuget/nuspec_file.png)  

Let's look at the .nuspec file that is generated:
``` xml
<?xml version="1.0"?>
<package >
  <metadata>
    <id>$id$</id>
    <version>$version$</version>
    <title>$title$</title>
    <authors>$author$</authors>
    <owners>$author$</owners>
    <licenseUrl>http://LICENSE_URL_HERE_OR_DELETE_THIS_LINE</licenseUrl>
    <projectUrl>http://PROJECT_URL_HERE_OR_DELETE_THIS_LINE</projectUrl>
    <iconUrl>http://ICON_URL_HERE_OR_DELETE_THIS_LINE</iconUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>$description$</description>
    <releaseNotes>Summary of changes made in this release of the package.</releaseNotes>
    <copyright>Copyright 2014</copyright>
    <tags>Tag1 Tag2</tags>
  </metadata>
</package>
```

First thing you'll notice is the variable names that start and end with a $ sign. In the next step, when we generate the actual package, the package generator will be told to grab these metadata from the assembly information. 

Here is how the assembly information of Nfield.SDK looks like:

![Nfield SDK Assembly info](/assets/NfieldSDK_Nuget/Nfield.SDK_AssemblyInfo.png)

Other fields which don't contain variable names need to be filled in before generating the package. Here is the .nuspec file after I edited all these fields:

``` xml
<?xml version="1.0"?>
<package >
  <metadata>
    <id>$id$</id>
    <version>$version$</version>
    <title>$title$</title>
    <authors>$author$</authors>
    <owners>$author$</owners>
    <licenseUrl>http://www.gnu.org/licenses</licenseUrl>
    <projectUrl>https://github.com/NIPOSoftware/Nfield-SDK</projectUrl>
    <iconUrl></iconUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>$description$</description>
    <releaseNotes></releaseNotes>
    <copyright>Copyright © NIPO Software 2013</copyright>
    <tags>nfield marketresearch capi cawi</tags>
  </metadata>
</package>
```
<br>
### 3. Creating the actual nuget package (.nupkg) for Nfield.SDK

<br>

``` bash executing the nuget package command for Nfield.SDK in release mode
nuget pack Nfield.SDK.csproj -Prop Configuration=Release
```

Notice that "nuget pack" command is not executed on the .nuspec file but on the .csproj file. But according to the nuget documentation this command picks up both the .nuspec file and the .csproj file from the directory. After executing the pack command here is what I got:

![Nfield SDK - Icon Url Can't be empty](/assets/NfieldSDK_Nuget/Icon_URL_CanNotBeEmpty.png)

For a moment, I thought ignoring the iconUrl entry in the .nuspec file would not be a big deal but I was wrong.

This is what [.nuspec reference]() says about the iconUrl:
>A URL for the image to use as the icon for the package in the Manage NuGet Packages dialog box. This should be a 32x32-pixel .png file that has a transparent background.

I immediately headed to http://nfieldmr.com/ to steal an nfield icon and indeed I found one. I shrinked the size to 32x32 and then committed it to the root of Nfield.SDK github repository.. hoping to be able to get a public URL for it afterwards. And voila! check the image below. Clicking the "Raw" button gives me this URL for my new icon: https://raw.githubusercontent.com/NIPOSoftware/Nfield-SDK/development/icon-nfield.png

![Nfield icon on github](/assets/NfieldSDK_Nuget/nfield-icon-on-github.png)

So after putting the icon URL into the .nuspec file here is what I have.. the final version of the .nuspec file:

``` xml
<?xml version="1.0"?>
<package >
  <metadata>
    <id>$id$</id>
    <version>$version$</version>
    <title>$title$</title>
    <authors>$author$</authors>
    <owners>$author$</owners>
    <licenseUrl>http://www.gnu.org/licenses</licenseUrl>
    <projectUrl>https://github.com/NIPOSoftware/Nfield-SDK</projectUrl>
    <iconUrl>https://raw.githubusercontent.com/NIPOSoftware/Nfield-SDK/development/icon-nfield.png</iconUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>$description$</description>
    <releaseNotes></releaseNotes>
    <copyright>Copyright © NIPO Software 2013</copyright>
    <tags>nfield marketresearch capi cawi</tags>
  </metadata>
</package>
```

After running the same nuget pack command one more time, the package is successfully created.

![Nfield SDK - Successful Package Creation 1](/assets/NfieldSDK_Nuget/SuccessfulPackageCreation_1.png)

And here is how the package (.nupkg) looks like in the library folder:

![Nfield SDK - Successful Package Creation 2](/assets/NfieldSDK_Nuget/SuccessfulPackageCreation_2.png)

### 4. Publishing the Nfield.SDK package to nuget

Now I have my nuget package file waiting to be published. Or wait... maybe I should test it before publishing it to the public NuGet repository. There is a way to create a local package repository on the local file system of the machine and test the package using this local nuget repository. But because this blog post is getting longer than I expected I will save it for another day.

Let's assume that I've tested the Nfield.SDK nuget package and I'm ready to publish it. First thing that is necessary is to get the long due NIPO Software account on nuget.org.

Checkout the screenshot below from nuget.org account page of NIPO Software. Two big buttons on the first row tells me that I can upload packages using nuget.org and I can also manage my existing packages. The other interesting information is the "API Key" at the bottom. This is a GUID that can be used from the nuget command utility. Using this key it's possible to publish nuget packages from the command line without signing in to nuget.org.

![Nipo Software NuGet Account](/assets/NfieldSDK_Nuget/NIPOSoftware-NuGet.png)

So this means it's possible to publish a nuget package both using nuget.exe from command line and using nuget.org. This is how nuget packages are pushed using nuget.exe:

``` bash publishing Nfield.SDK using nuget.exe
nuget SetApiKey NIPOSoftware-API-Key
nuget push Nfield.SDK.1.0.0.0.nupkg
```

I took the other approach and directly uploaded the package using nuget.org web interface. After I uploaded the "Nfield.SDK.1.0.0.0.nupkg" file, nuget.org asked me to verify the details of the package before clicking publish the last time.

![Nfield SDK - Verify NuGet Publish](/assets/NfieldSDK_Nuget/VerifyingUpload_RightBeforePublish.png)

All seems fine for now and I click publish. The package is published for the world to see. Below you see the Nfield.SDK page from the NuGet Gallery. Using "Install-Package Nfield.SDK" command from the VS Package Manager Console will install Nfield SDK to your project. Also notice near the bottom there is a section called "Dependencies". Nfield.SDK depends on one other nuget package and it's visible here. This was automatically handled by nuget when I created the package using the .csproj file. 

![Nfield SDK - Successfully uploaded to Nuget](/assets/NfieldSDK_Nuget/SuccessfullyUploadedNfield.SDK.png)

So now as a final step, let's go to Visual Studio and find Nfield.SDK in package manager. Searching for Nfield brings me the Nfield.SDK package. Here is a screenshot right before I installed Nfield.SDK for my Nfield test project which I will discuss in my next blog post.

![Nfield SDK - Online in NuGet](/assets/NfieldSDK_Nuget/NfieldSDK_Online_InNuget.png)

Now that Nfield.SDK is online on NuGet, I can easily install and access it from my own projects. In the next blog post I'll show you how you can install Nfield.SDK and use it to do something useful - like adding an interviewer to your market research office.

Until then, take care.


