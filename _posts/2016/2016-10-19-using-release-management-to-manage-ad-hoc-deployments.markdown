---
layout: post
title: Using Release Management to Manage Ad-Hoc Deployments
date: '2016-10-19 12:08:56'
tags:
- releasemanagement
---

Release Management (RM) is awesome – mostly because it works off the amazing cross platform build engine. Also, now that [pricing is announced](https://blogs.msdn.microsoft.com/visualstudioalm/2016/09/26/pricing-for-release-management-in-tfs-15/), we know that it won’t cost an arm and a leg!

When I work with customers to adopt RM, I see two kinds of deployments: _repeatable_ and _ad-hoc_. RM does a great job at repeatable automation – that is, it is great at doing the same thing over and over. But what about deployments that are different in some way every time? Teams love the traceability that RM provides – not just the automation logs, but also the approvals. It would be great if you could track ad-hoc releases using RM.

## The Problem

The problem is that RM doesn’t have a great way to handle deployments that are slightly different every time. Let’s take a very typical example: ad-hoc SQL scripts. Imagine that you routinely perform data manipulation on a production database using SQL scripts. How do you audit what script was run and by whom? “We can use RM!” I hear you cry. Yes you can – but there are some challenges.

Ah-hoc means (in this context) _different every time_. That means that the script you’re running is bound to change every time you run the release. Also, depending on how dynamic you want to go, even the target servers could change – sometimes you’re executing against server A, sometimes against server B, sometimes both. “Just make the script name or server name a variable that you can change at queue time,” I hear you say. Unfortunately, unlike builds, you can’t specify parameter values at queue time. You _could_ create a release in draft and then edit the variables for that run of the release, but this isn’t a great experience since you’re bound to forget things – and you’ll have to do this every time you start a release.

## A Reasonable Solution

I was at a customer who were trying to convert to RM from a home-grown deployment tool. Besides “repeatable” deployments their tool was handling several hundred ad-hoc deployments a month, so they had to decide whether or not to keep the ad-hoc deployments in the home-grown tool or migrate to RM. So I mailed the Champs List – a mailing list direct to other MVPs and the VSTS Product Group in Microsoft (being an MVP has to have some benefits, right?) – and asked them what _they_ do for ad-hoc deployments. It turns out that they use ad-hoc deployments to turn feature switches on and off, and they run their ad-hoc deployments with RM – and while I didn’t get a lot of detail, I did get some ideas.

I see three primary challenges for ad-hoc release definitions:

1. What to execute

- Where does the Release get the script to execute? You could create a network share and put a script in there called “adhoc.sql” and get the release to execute that script every time it runs. Tracking changes is then a challenge – but we’re developers and already know how to track changes, right? Yes: _source control_. So source control the script – that way every time the release runs, it gets the latest version of the script and runs that. Now you can track what executed and who changed the script. And you can even perform code-review prior to starting the release – bonus!

1. What to execute

- Is there an echo here? Well no – it’s just that if the release is executing the same script every time, there’s a danger that it could well – execute the same script. That means you have to either write your script defensively – that is, in such a manner that it is idempotent (has the same result no matter how many times you run it) or you have to keep a record of whether or not the script has been run before, say using a table in a DB or an Azure table or something. I favor idempotent scripts, since I think it’s a good habit to be in when you’re automating stuff anyway – so for SQL that means doing “if this record exists, skip the following steps” kind of logic or using MERGE INTO etc.

1. Where to execute

- Are you executing against the same server every time or do the targets need to be more dynamic? There are a couple of solutions here – you could have a text doc that has a list of servers, and the release definition reads in the file and then loops, executing the script against the target servers one by one. This is dynamic, but dangerous – what if you put in a server that you don’t mean to? Or you could create an environment per server (if you have a small set of servers this is ok) and then set each environment to manual deployment (i.e. no trigger). Then when you’re ready to execute, you create the release, which just sits there until you explicitly tell it which environment (server) to deploy to.

## Recommended Steps

While it’s not trivial to set up an ad-hoc deployment pipeline in RM, I think it’s feasible. Here’s what I’m going to start recommending to my customers:

1. Create a Git repository with a well-defined script (or root script)
2. Create a Release that has a single artifact link – to the Git repo you just set up, on the master branch
3. Create an environment per target server. In that environment, you’re conceptually just executing the root script (this could be more complicated depending on what you do for ad-hoc deployments). All the server credentials etc. are configured here so you don’t have to do them every time. You can also configure approvals if they’re required for ad-hoc scripts. Here’s an example where a release definition is targeting (potentially) ServerA, ServerB and/or ServerC. This is only necessary if you have a fixed set of target servers and you don’t always know which server you’re going to target:
<!--kg-card-begin: html-->[![image](/assets/images/files/00d7b0aa-5514-4fc9-8441-91682c57fcad.png "image")](/assets/images/files/08f9fe08-c9f4-4008-8508-d20ed193a22e.png)<!--kg-card-end: html-->
1. Here I’ve got an example of copying a file (the root script, which is in a Git artifact link) to the target server and then executing the script using the WinRM SQL task. These tasks are cloned to each server – of course the server name (and possibly credentials) are different for each environment – but you only have to set this up once.
2. Configure each environment to have a manual trigger (under deployment conditions). This allows you to select which server (or environment) you’re deploying to for each instance of the release:
<!--kg-card-begin: html-->[![image](/assets/images/files/fea0da76-88e4-4e94-9e3c-a2a015872a3e.png "image")](/assets/images/files/4e5b20a8-69fc-487f-b6bb-a82bb525b9e3.png)<!--kg-card-end: html-->
1. Enable a branch policy on the master branch so that you have to create a Pull Request (PR) to get changes into master. This forces developers to branch the repo, modify the script and then commit and create a PR. At that point you can do code review on the changes before queuing up the release.
2. When you’ve completed code review on the PR, you then create a release. Since all the environments are set to manual trigger, you now can go and manually select which environment you want to deploy to:
<!--kg-card-begin: html-->[![image](/assets/images/files/7e949357-3b37-40a1-b5e9-22b61cecdc8e.png "image")](/assets/images/files/9517b602-152c-4ed7-9df0-2520eba8c400.png)<!--kg-card-end: html-->
1. Here you can see how the status on each environment is “Not Deployed”. You can now use the deploy button to manually select a target. You can of course repeat this if you’re targeting multiple servers for this release.

## Conclusion

With a little effort, you can set up an ad-hoc release pipeline. This gives you the advantages of automation (since the steps and credentials etc. are already set up) as well as auditability and accountability (since you can track changes to the scripts as well as any approvals). How do you, dear reader, handle ad-hoc deployments? Sound off in the comments!

Happy releasing!

