---
layout: post
title: DevOps Drives Better Architecture–Part 1 of 2
date: '2017-02-19 05:05:42'
tags:
- devops
---

(Read [part 2](http://bit.ly/devopsarch2) here)

I haven’t blogged for a long while – it’s been a busy few months!

One of the things I love about being a DevOps consultant is that I have to be technically proficient – I can’t help teams develop best practices if I don’t know the technology to at least a reasonable depth – but I also get to be a catalyst for change. I love the cultural dynamics of DevOps. After all, as my friend Donovan Brown says, “DevOps is the union of _people_, processes and tools…”. When you involve people, then you get to watch (or, in my case, influence) culture. And it fascinates me.

I only recently read [Continuous Delivery](https://www.amazon.com/Continuous-Delivery-Deployment-Automation-Addison-Wesley/dp/0321601912/ref=asap_bc?ie=UTF8) by [Jez Humble](https://twitter.com/jezhumble) and [David Farley](http://www.davefarley.net/). I was pleased at how much of their insights I’ve been advocating “by instinct” over my years of ALM and DevOps consulting. Reading their book sparked the thoughts that I’ll put into this two-part post.

This part will introduce the thought that DevOps and architecture are symbiotic – good architecture makes for good DevOps, and good DevOps drives good architecture. Ill look at Builds and Source Control in particular. In part 2, I’ll discuss infrastructure as code, database design, automated testing and monitoring and how they relate to DevOps and vice versa.

## Tools, tools, tools

Over the past few months, the majority of my work has been to help teams implement Build/Release Pipelines. This seems inevitable to me given the state of DevOps in the market in general – most teams have made (or are in the process of making) a shift to agile, iterative frameworks for delivering their software. As they get faster, they need to release more frequently. And if they’ve got manual builds and deployments, the increasing frequency becomes a frustration because they can’t seem to deploy fast enough (or consistently enough). So teams are starting to want to automate their build/release flows.

It’s natural, therefore, to immediately look for a tool to help automation. And for a little help from your friends at Northwest Cadence to help you do it right!

Of course my tool of choice for build/release pipelines is Visual Studio Team Services (VSTS) or Team Foundation Server (TFS) for a number of reasons:

1. The build agent is cross platform (it’s built on .NET Core, so runs wherever .NET Core runs)
2. The build agent is also the release agent
3. The build agent can run tests
4. The task-based system has a good Web-based UI, allowing authoring from wherever you have a browser
5. The logging is great – allowing fast debugging of build issues
6. Small custom logic can easily be handled with inline scripts
7. If you can script it, the agent can do it – whether it’s bat, ps1 or sh
8. Extensions are fairly easy to create
9. There is a large and thriving marketplace for extensions

## Good Architecture Means Easier DevOps

Inevitably implementing build automation impacts how you organize your source control. And implementing a release pipeline impacts how you test. And implementing continuous deployment impacts IT, since there’s suddenly a need to be able to spin up and configure and tear down environments on the fly. I love seeing this progression – but it’s often painful for the teams I’m working with. Why? Because teams start realizing that if their architecture was better, it would make other parts of the DevOps pipeline far easier to implement.

For example, if you start automating releases, pretty soon you start wanting to run automated tests since your tests start becoming the bottleneck to delivery. At this point, if you’ve used good architectural principles like interfaces and inversion of control, writing unit tests is far easier. If you haven’t, you have a far harder time writing tests.

Good architecture can make DevOps easier for you and your team. We’ve been told to do these things, and often we’ve found reasons not to do them (“I don’t have time to make an interface for everything!” or “I’ll refactor that class to make it more testable in the next sprint” etc. etc.). Hopefully I can show you how these architectural decisions, if done with DevOps in mind, will not only make your software better but help you to implement better DevOps, more easily!

## The Love Triangle: Source Control, Branches and Builds

I really enjoy helping teams implement their first automated builds. Builds are so foundational to good DevOps – and builds tend to force teams to reevaluate their code layout (structure), dependencies and branching strategy.

Most of the time, the teams have their source code in some sort of source control system. Time and time again, the teams that have a good structure and simple branching strategies have a far easier time getting builds to work well.

Unfortunately, most repositories I look at are not very well structured. Or the branches represent environments so you see MAIN/DEV/PROD (which is horrible even though most teams don’t know why – read on if this is you). Or they have checked binaries into source control instead of using a package manager like NuGet. Or they have binary references instead of project references.

Anyway, as we get the build going, we start to uncover potential issues most teams don’t even know they have (like missing library references or conflicting package versions). After some work and restructuring, we manage to get a build to compile. Whoop!

### Branching

After the initial elation and once the party dies down, we take a look at the branching strategy. “We need a branch for development, then we promote to QA, and once QA signs off we promote to PROD. So we need branches for each of these environments, right?” This is still a very pervasive mindset. However, DevOps – specifically release pipelines – should operate on a simple principle: build once, deploy many times. In other words, the bits that you deploy to DEV should be the same bits that you deploy to QA and then to PROD. Don’t just take my work for it: read Continuous Delivery – the authors emphasize this over and over again!. You can’t do that if you have to merge and build each time you want to promote code between environments. So how do you track what code is where and promote parallel development?

### Builds and Versioning

I advocate for a master/feature branch strategy. That is, you have your stable code on master and then have multiple feature branches (1 to n at any one time) that developers work on. Development is done on the feature branch and then merged into master via Pull Request when it’s ready. At that point, a build is queued _which versions the assemblies and tags the source with the version_ (which is typically the build number).

That’s how you keep track of what code is where – by versions and tags that your build keeps the keys for. That way, you can do hotfixes directly onto master even if you’ve already merged code that is in the pipeline and not yet in production. For example, say you have 1.0.0.6 in prod and you merge some code in for a new feature. The build kicks in and produces version 1.0.0.7 which gets automatically deployed to the DEV environment for integration testing. While that’s going on, you get a bug report from PROD. Oh no! We’ve already merged in code that isn’t yet in PROD, so how do we fix it on master?!?

It’s easy – we know that 1.0.0.6 is in PROD, so we branch the code using tag 1.0.0.6 (which the build tagged in the repo when it ran the 1.0.0.6 build). We then fix the issue in the branch build off of this branch. A new build – 1.0.0.8. We take a quick look at this and fast-track it through until it’s deployed and business can continue. In the meantime, we can abandon the 1.0.0.7 build that’s currently in the deployment pipeline. We merge the hotfix branch back to the master and do a new build – 1.0.0.9 that now has the hotfix as well as the new feature. No sweat.

“Hang on. Pull Requests? Feature branches? That sounds like Git.” Yes, it does. If you’re not on Git, then you’d better have a convincing reason not to be. You can do a lot of this with TFVC, but it’s just harder. So just get to Git. And as a side benefit, you’ll get a far richer code review experience (in the form of Pull Requests) so your quality is likely to improve. And merging is easier. And you can actually cherry-pick. I could go on and on, but there’s enough Git bigots out there that I don’t need to add my voice too. But get to Git. Last word. Just Do It.

## Small Repos, Microservices and Package Management

So you’re on Git and you have a master/feature branch branching strategy. But you have multiple components or layers and you need them to live together for compilation, so you put them all into one repo, right? Wrong. You need to separate out your components and services into numerous small repos. Each repo should have Continuous Integration (CI) on it. This change forces teams to start decomposing their monolithic apps into shared libraries and microservices. “Wait – what? I need to get into microservices to get good DevOps?” I hear you yelling. Well, another good DevOps principle is releasing small amounts of change often. And if everything is mashed together in a giant repo, it’s hard to do that. So you need to split up your monoliths into smaller components that can be independently released. Yet again, good architecture (loose coupling, strict service boundaries) promotes good DevOps – or is it DevOps finally motivating you to Do It Right™ like you should have all along?

You’ve gone ahead and split out shared code and components. But now your builds don’t work because your common code (your internal libraries) are suddenly in different repos. Yes, you’re going to need some package management tool. Now, as a side benefit, teams can opt-in to changes in common libraries rather than being forced to update project references. This is a great example of how good DevOps influences good development practices! Even if you just use a file share as a NuGet source (you don’t necessarily need a full-blown package management system) you’re better off.

## Conclusion

In this post, we’ve looked at how good source control structure, branching strategies, loosely coupled architectures and package management can make DevOps easier. Or perhaps how DevOps pushes you to improve all of these. As I mentioned, good architecture and DevOps are symbiotic, feeding off each other (for good or bad). So make sure it’s for good! Now go and read [part 2](http://bit.ly/devopsarch2) of this post.

Happy architecting!

