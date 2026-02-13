---
layout: post
title: Updating XAML Release Builds after Upgrading Release Management Legacy from
  2013 to 2015
date: '2016-05-17 21:13:18'
tags:
- releasemanagement
---

You need to get onto the new [Release Management](https://msdn.microsoft.com/library/vs/alm/release/overview) (the web-based one) in VSTS or TFS 2015 Update 2. The new version is far superior to the old version for numerous reasons – it uses the new Team Build cross-platform agent, has a much simpler UI for designing releases, has better logging etc. etc.

However, I know that lots of teams are invested in [Release Management “legacy”](https://www.visualstudio.com/en-us/get-started/release/rm-for-vs2015-vs). Over the weekend I helped a customer upgrade their TFS servers from 2013 to 2015.2.1. Part of this included upgrading their Release Management Server from 2013 to 2015. This customer has been using Release Management since it was still InRelease! They have a large investment in their current release tools, so they need it to continue working so that they can migrate over time.

The team also [trigger releases](https://msdn.microsoft.com/library/vs/alm/release/previous-version/trigger-a-release) in Release Management from their XAML builds. Unfortunately, their builds started breaking once we upgraded the Release Management client on the build servers. The build error was something like: “Invalid directory”. (Before upgrading the client, the release step failed saying that the build service needed to be set up as a Service User – which it was. This error is misleading – it’s an indication that you need to upgrade the RM Client on the build machine).

### Upgrading XAML Build Definitions

It turns out that the Release Management XAML templates include a step that reads the registry to obtain the location of the Release Management client binaries. This registry key has changed from RM 2013 to 2015, so you have two options:

1. If you used the older ReleaseGitTemplate.12.xaml or ReleaseTfvcTemplate12.xaml files from RM 2013, then you can replace them with the updated release management templates that ship with Release Management client (find them in **\Program Files (x86)\ Microsoft Visual Studio 14.0\ReleaseManagement\bin**)
2. If you customized your own templates (or customized the RM 2013 templates), you need to update your release template

Fortunately updating existing templates to work with the new RM client is fairly trivial. Here are the steps:

1. Check out your existing XAML template
2. Open it in Notepad (or using the XML editor in VS)
3. Find the task with DisplayName “Get the Release Management install directory”. One of the arguments is a registry key – it will be something like **HKEY\_LOCAL\_MACHINE\Software\Microsoft\ReleaseManagement\12.0\Client\.** Replace this key with this value: **HKEY\_LOCAL\_MACHINE\Software\WOW6432Node\Microsoft\ReleaseManagement\14.0\Client\**
4. The task just below is for finding the x64 directory – you can do the same replacement in this task.
5. Commit your changes and checkin
6. Build and release
7. Party

Thanks to Jesse Arens for this great find!

On a side note – the ALM Rangers have a project that will help you port your “legacy” RM workflows to the new web-based releases. You can find it [here](https://github.com/ALM-Rangers/Migrate-assets-from-RM-server-to-VSTS).

Happy releasing! (Just move off XAML builds and Release Management legacy as soon as possible – for your own sanity!)

