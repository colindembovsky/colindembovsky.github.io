---
layout: post
title: Automated Builds–Why They’re Absolutely Essential (Part 1)
date: '2013-06-18 19:23:00'
tags:
- build
- alm
---

A couple of weeks ago I was doing a road-show where I did demos of TFS 2012 and it’s capabilities. I do a 4 hour demo that shows an end-to-end scenario, showing capabilities such as requirements management and elicitation, work management, developer tools, quality tools, testing tools, automated builds, lab management and reporting all using TFS. I visited 9 different companies, and most of them asked, “Why should we do builds?” This is something I want to address – you need to be doing builds, and you need to understand why they are so key to successful and effective software development. Builds are no longer an optional extra!

## General Reasons to do Automated Builds

Before I dive into doing automated builds using TFS Team Build, there are two general principles that apply to doing automated builds that I’d like to unpack: _Consistency_ and _Quality_. These principles transcend the choice of build engine – be it TFS Build (or TeamBuild), Hudson, Cruise Control, Jenkins or any other engine.

<u>1. Consistency</u>

How do you deploy your code into test (or for that matter, into production)? Most teams start off using the “Publish” menu option from within Visual Studio, either publishing directly to test (or even production) environments, or publishing locally and copying to the target servers. Let me be brutally honest – this is simply immature. This “deploy from Visual Studio” is really something only amateur developers and hobbyists should be using. Why? Because there is so much risk attached to doing things this way.

Some teams argue that they only do this to “test” environments and start getting “serious” only when deploying to production. I argue that you should treat even your test environments as if they were production targets, so that you get the deployment process right there before promoting to production. If you can do it right in pre-production environments, chances are you’ll be able to do it right in production.

Let’s look at some of the risks of “Publish from VS” deployments:

1. You could be deploying anything
2. How do you know that you’ve only got the code that’s in Source Control? Perhaps you opened your solution and fiddled around a bit, just to try an idea. Now you’ve published untested code into production.
3. You could have dependencies on anything
4. Let’s imagine you’ve got a dependency on some 3rd party library – Enterprise Library or Entity Framework or MVC or any other library. Your target server has version X installed, but you’re a dev, so you install version Y (you want to be on the latest and greatest, right?). Now you’ve compiled and deployed against the wrong version.
5. Your “guy/gal who deploys” gets hit by a bus
6. Most times, a senior team member is doing the deployment. Now all your deployment know-how is invested in one person – what if that person is out sick or his/her machine crashes? Now you can’t deploy.
7. Even if you spread the deployment pain knowledge around, how long is the deploy-er going to spend doing this deployment? In the best case scenario, a few minutes. Generally though, this takes a lot longer. Do you really want a senior team member indisposed for hours and hours every month to do deployment?
8. You can’t “do it again” (in most cases)
9. What if your deploy-er is having a rough day and forgets to copy this folder or forgets to add that bit of new config to the existing configuration files? Manual deployments are not repeatable, so they’re error prone.

So what does this have to do with automated builds? Automated builds address each of the above risks. Fewer risks translates into better quality and higher productivity.

1. You’re deploying “known” code
2. Since the build engine is going to check out the latest version of source control, you know exactly what code is going to be compiled. No wondering if there are any “unintentional experiments that I forgot to delete”.
3. You control dependencies on the build server
4. You’re going to configure your build server – and that means it will have only what you install (assuming you lock it down). No rogue libraries or experimental versions – just exactly what you need to get into production.
5. Anyone (who has permission) can trigger the build
6. Since you’re setting up an automated build process, you’ll be able to trigger it with a single-click. No need for you to designate a deploy-er who has a whole lot of knowledge about what to compile. Once your build is set up, you can “just do it” again and again.
7. Also, since it’s automated, you can “fire and forget”. Even if the build takes half an hour, your team lead is free to continue coding or whatever you really pay him/her to do (as opposed to watching compilation and copying files all over the place).
8. Builds are repeatable
9. Most build engines can “re-build” – do an exact build again. Since the process is automated, you can rest assured that no step is going to be forgotten by mistake.

<u>2. Quality</u>

Quality is something that’s hard to measure. Let’s consider an example. If a user expects the system to save a record to the database when they click the “Save” button, and it works, then quality is high, right? Not necessarily. Perhaps the “Save” operation only works when the information is “clean” – and breaks if the data is invalid (perhaps it should warn the user that there is invalid data?).

Automated builds go some way to providing a _measure of quality_. How do you know that the code you are publishing from VS is good code? What measures do you have to even assess this? I argue that if you don’t have automated builds (with unit tests – something we’ll again discuss in a later post) then you’ve got no objective measure.

Let’s consider some of the advantages that automated builds bring in terms of quality:

1. Packaging your binaries
2. Let’s say you want to test your code in Staging before you deploy to Production. If you publish from VS, then you’ll be doing the publish twice – once for Staging, and then once for Production. Automated builds give you the advantage of producing one package (be that a WebDeploy package or installer or whatever) that you can deploy multiple times, knowing that you’re deploying exactly the same thing every time.
3. Quality Measurement – i.e. test statistics
4. I’ll discuss unit testing in another post – but assume that you have unit tests (and you’re analysing code coverage). If you don’t have a build of some sort, how do you know that the tests all passed? You’d have to take the word of your deploy-er. With automated builds, you can look at the build reports to see test pass/fail rates as well as coverage. If there are failing tests or coverage is too low, you block deployment. Also, if your build engine is putting these measurements into some sort of database you’ll be able to track quality trends over time.
5. Removing user error
6. A good automated build process is exactly that – _automated_. That means that the process can’t “forget” to link to some library or to run this or that test. This means better quality.
7. Definition of Done
8. Just because a developer says it’s done, it usually means that “it’s sort-of-nicely-coded-and-works-on-my-machine”. An automated build will run unit tests (the first of a few quality gates that should be part of your process). Then you should be deploying this build out to a pre-production environment. Testers (or at least “Power” users) should then manually test the build. Only once they sign off should the build be deployed out to Production. Since an automated build has produced the package, you know that the same package (that’s now passed automated and manual tests) is going out to Production.
9. All of this means you get a standard, repeatable and consistent process that can become part of your “definition of done”. If it doesn’t pass unit tests, block it. If coverage is too low, block it. If it doesn’t pass manual testing, block it. Only once the build has passed these gates can it go to Production.

## Consider the Cost

So let’s now ask which process would let you sleep easier the night of your rollout to Production? The one where a developer claims the unit tests are passing with sufficient coverage and does a “Publish” from Visual Studio, or the one where you’ve got an automated build report showing test success and coverage, approval from testers that the build passed manual tests, and a script that’s proven itself over and over for deployment?

I’d like to end with this consideration: the “earlier” in the process you find a bug, the cheaper it is to fix. Consider finding a bug while you’re coding. You fix it then-and-there: cost to company is a couple minutes of your time, so probably a few cents. On the other end of the spectrum, consider cost to company of a bug in Production: in terms of pure development, there’s the time of the person who finds the bug – then the time of the call-centre that they call, sometimes the ops team, 2nd line support, then some developer who spends a few hours trying to reproduce the issue and eventually fix it. Then it has to go through testing etc. etc. – and that’s just in terms of “direct” costs. Bugs in Production could cause other costs such as financial costs for legal errors or even reputational cost for your company.

The bottom line is this: investing some time now to automate builds (and add unit tests) will save you lots of time and money in production and operational issues. It’s a fact.

In the [next post](http://www.colinsalmcorner.com/2013/06/automated-buildswhy-theyre-absolutely_18.html), I’ll talk about the Team Build automation engine.

Happy building!

