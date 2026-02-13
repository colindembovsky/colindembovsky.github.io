---
layout: post
title: Building VS 2015 Setup Projects in Team Build
date: '2016-01-13 05:13:20'
tags:
- build
---

Remember when Visual Studio had a setup project template? And then it was removed? Then you moved to [WiX](http://wixtoolset.org/) and after learning it for 3 months and still being confused, you just moved to Web Apps?

Well everyone complained about the missing setup project templates and MS finally added it back in as an [extension](https://visualstudiogallery.msdn.microsoft.com/f1cc3f3e-c300-40a7-8797-c509fb8933b9). Which works great if you build out of Visual Studio – but what about automated builds? Turns out they don’t understand the setup project, so you have to do some tweaking to get it to work.

## Setup Project Options

There are a couple of options if you’re going to use setup projects.

1. [ClickOnce](https://msdn.microsoft.com/en-us/library/t71a733d.aspx). This is a good option if you don’t have a deployment solution that can deploy new versions of your application (like System Center or the like). It requires fudging on the builds to get versioning to work in some automated fashion. At least it’s free.
2. [WiX](http://wixtoolset.org/). Free and very powerful, but really hard to learn and you end up programming in XML – which is a pain. However, if you need your installer to do “extra” stuff (like create a database during install) then this is a good option. Automation is also complicated because you have to invoke Candle.exe and Light.exe to “build” the WiX project.
3. VS Setup Projects. Now that they’re back in VS, you can use these projects to create installers. You can’t do too much crazy stuff – this just lays down the exe’s and gets you going. It’s easy to maintain, but you need to tweak the build process to build these projects. Also free.
4. [InstallShield](http://www.flexerasoftware.com/producer/products/software-installation/installshield-software-installer/) and other 3rd party paid installer products. These are typically powerful, but expensive. Perhaps the support you get is worth the price, but you’ll have to decide if the price is worth the support and other features you don’t get from the other free solutions.

## Tweaking Your Build Agent

You unfortunately won’t be able to build setup projects on the Hosted build agent because of these tweaks. So if you’ve got a build agent, here’s what you have to do:

1. Install Visual Studio 2015 on the build machine.
2. Install the [extension](https://visualstudiogallery.msdn.microsoft.com/f1cc3f3e-c300-40a7-8797-c509fb8933b9) onto your build machine.
3. Configure the build agent service to run under a known user account (not local service, but some user account on the machine).
4. Apply a registry hack – you have to edit HKCU\SOFTWARE\Microsoft\VisualStudio\14.0\_Config\MSBuild\EnableOutOfProcBuild to have a DWORD of 0 (I didn’t have the key, so I just added it). If you don’t do this step, then you’ll probably get an obscure error like this: “ERROR: An error occurred while validating. &nbsp;HRESULT = '8000000A'”
5. Customize the build template (which I’ll show below).

It’s fairly nasty, but once you’ve done it, your builds will work without users having to edit the project file or anything crazy.

## Customizing the Build Definition

You’ll need to configure the build to compile the entire solution first, and then invoke Visual Studio to create the setup package.

Let’s walk through creating a simple build definition to build a vdproj.

1. Log in to VSTS or your TFS server and go to the build hub. Create a new build definition and select the Visual Studio template. Select the source repo and set the default queue to the queue that your build agent is connected to.
2. Just after the Visual Studio Build task, add a step and select the “Command Line” task from the Utility section.
3. Enter the path to devenv.com for the Tool parameter (this is typically “C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\devenv.com”).
4. The arguments have the following format: _solutionPath_ /build _configuration_ _projectPath_
5. solutionPath is the path to the solution file
6. configuration is the config (debug, release etc.)
7. projectPath is the path to the vdproj file
8. Finally, expand the “Advanced” group and set the working folder to the path of the sln file and check the “Fail on Standard Error” checkbox.

Here’s an example:

<!--kg-card-begin: html-->[![image](/assets/images/files/4ceb0068-6af9-4b28-947a-21de05cc1a69.png "image")](/assets/images/files/34c056cc-67e1-43dd-99b7-3271105b883c.png)<!--kg-card-end: html-->

For reference, here’s how my source is structured:

<!--kg-card-begin: html-->[![image](/assets/images/files/4375ef99-63ed-4533-ad01-3bf743d77616.png "image")](/assets/images/files/fcd94210-91dd-43e5-8209-6237066cd7d6.png)<!--kg-card-end: html-->

You can then publish the setup exe or msi if you need to. You can run tests or scripts or anything else during the build (for ease I delete the unit test task in the above example).

I now have a successful build:

<!--kg-card-begin: html-->[![image](/assets/images/files/a35d538f-a743-487b-a9c4-aca62c8549f5.png "image")](/assets/images/files/d9f53d62-d230-442e-b917-7bd29dbf2e0b.png)<!--kg-card-end: html-->

And the msi is in my drop, ready to be deployed in Release Management:

<!--kg-card-begin: html-->[![image](/assets/images/files/2ad1710a-8b51-47ad-b25f-3f837c2177dc.png "image")](/assets/images/files/a1613f42-5c56-4643-8f8a-32bc4fdd3cde.png)<!--kg-card-end: html-->

Happy setup building!

