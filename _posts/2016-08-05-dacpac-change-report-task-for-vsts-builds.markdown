---
layout: post
title: DacPac Change Report Task for VSTS Builds
date: '2016-08-05 22:57:24'
tags:
- build
---

Most development requires working against some kind of database. Some teams choose to use Object Relational Mappers (ORMs) like Entity Framework. I think that should be the preferred method of dealing with databases (especially code-first), but there are times when you just have to work with a database schema.

Recently I had to demo [ReadyRoll](https://www.red-gate.com/products/sql-development/readyroll/?gclid=CJDDxZ7Nqs4CFbEV0wodFIUCJA) in VSTS. I have to be honest that I don’t like the paradigm of ReadyRoll – migration-based history seems like a mess compared to model-based history (which is the approach that SSDT takes). That’s a subject for another post (some day) or a discussion over beers. However, there was one thing that I really liked – the ability to preview database changes in a build. The [ReadyRoll extension](https://marketplace.visualstudio.com/items?itemName=redgatesoftware.redgate-readyroll) on the VSTS marketplace allows you to do just that.

So I stole the idea and made a task that allows you to see SSDT schema changes from build to build.

## Using the Task

Let’s consider the scenario: you have an SSDT project in source control and you’re building the dacpac in a Team Build. What the task does is allow you to see what’s changed from one build to the next. Here’s what you need to do:

1. Install [Colin’s ALM Corner Build Tasks Extension](https://marketplace.visualstudio.com/items?itemName=colinsalmcorner.colinsalmcorner-buildtasks) from the VSTS Marketplace
2. Edit the build definition and go to Options. Make sure “Allow Scripts to Access OAuth Token” is checked, since the task requires this. (If you forget this, you’ll see 403 errors in the task log).
3. Make sure that the dacpac you want to compare is being published to a build drop.
4. Add a “DacPac Schema Compare” task

That’s it! Here’s what the task looks like:

<!--kg-card-begin: html-->[![image](/assets/images/files/ece36d55-850e-4d5b-9b6b-b61d9c2d5700.png "image")](/assets/images/files/a86743a3-166a-4194-8981-d209dad9f5cc.png)<!--kg-card-end: html-->

Enter the following fields:

1. The name of the drop that your dacpac file is going to be published to. The task will look up the last successful build and download the drop in order to get the last dacpac as the source to compare.
2. The name of the dacpac (without the extension). This is typically the name of the SSDT project you’re building.
3. The path to the compiled dacpac for this build – this is the target dacpac path and is typically the bin folder of the SSDT project.

Now run your build. Once the build completes, you’ll see a couple new sections in the Build Summary:

<!--kg-card-begin: html-->[![image](/assets/images/files/b8aba5df-ee08-4199-b335-c31f88029ddf.png "image")](/assets/images/files/fa4526a0-91ef-4038-a50b-41ac2397b558.png)<!--kg-card-end: html-->

The first section shows the schema changes, while the second shows a SQL-CMD file so you can see what would be generated by SqlPackage.exe.

Now you can preview schema changes of your SSDT projects between builds! As usual, let me know here, on Twitter or on Github if you have issues with the task.

Happy building!

