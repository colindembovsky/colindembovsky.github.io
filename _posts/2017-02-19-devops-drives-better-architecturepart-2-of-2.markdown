---
layout: post
title: DevOps Drives Better Architecture–Part 2 of 2
date: '2017-02-19 05:06:13'
tags:
- devops
---

In [part 1](http://bit.ly/devopsarch1) I introduced some thoughts as to how good architecture makes DevOps easier. And how good DevOps drives better architecture – a symbiotic relationship. I discussed how good source control structure, branching strategies, loosely coupled architectures and package management can make DevOps easier. In this post I’ll share some thoughts along the same lines for infrastructure as code, database design and management, monitoring and test automation.

## Infrastructure as Code

Let’s say you get your builds under control, you’re versioning, you get your repos sorted and you get package management in place. Now you’re starting to produce lots and lots of builds. Unfortunately, a build doesn’t add value to anyone until it’s in production! So you’ll need to deploy it. But you’ll need infrastructure to deploy to. So you turn to IT and fill in the forms and wait 2 weeks for the servers…

Even if you can quickly spin up tin (or VMs more likely) or you’re deploying to PaaS, you still need to handle configuration. You need to configure your servers (if you’re on IaaS) or your services (on PaaS). You can’t afford to do this manually each time you need infrastructure. You’re going to need to be able to automatically spin up (and down) resources when you need them.

### Spinning Up From Scratch

I was recently at a customer that were using AWS VMs for their DevTest infrastructure. We were evaluating if we could replicate their automated processes in Azure. The problem was they hadn’t scripted creation of their environments _from scratch_ – they would manually configure a VM until it was “golden” and then use that as a base image for spinning up instances. Now that we were wanting to change their cloud host, they couldn’t do it easily because someone would have to spin up an Azure VM and manually configure it. If they had rather used the principle of scripting and automating configuration, we could have used the existing scripts to quickly spin up test machines on any platform or host. Sometimes you don’t know you need something until you actually need it – so do the right things early and you’ll be better off in the long run. Get into the habit of configuring via code rather than via UI.

### Deploy Whole Components – Always

In Continuous Delivery, Humble and Farley argue that it’s easier to deploy your entire system each time than trying to figure out what the delta is and only deploy that. If you craft your scripts and deployments so that they are _idempotent_, then this shouldn’t be a problem. Try to prefer declarative scripting (such as PowerShell DSC) over imperative scripting (like pure PowerShell). Not only is it easier to “read” the configuration, but the system can check if a component is in the required state, and if it is, just “no-op”. Make sure your scripts work irrespective of the initial state of the system.

If you change a single class, should you just deploy that assembly? It’s far easier to deploy the entire component (be that a service or a web app) than trying to work out what changed and what need to be deployed. Tools can also help here – web deploy, for example, only deploys files that are different. You build the entire site and it calculates at deployment time what the differences are. Same with SDDT for database schema changes.

Of course, getting all the requirements and settings correct in a script in source control is going to mean that you need to cooperate and team up with the IT guys (and gals). You’re going to need to work together to make sure that you’re both happy with the resulting infrastructure. And that’s good for everyone.

## Good Database Design and Management

### Where does your logic live?

How does DevOps influence database design and management? I used to work for a company where the dev manager insisted that all our logic be in stored procedures. “If we need to make a change quickly,” he reasoned, “then we don’t need to recompile, we can just update the SP!” Needless to say, our code was virtually untestable, so we just deployed and hoped for the best. And spent a lot of time debugging and fighting fires. It wasn’t pretty.

Stored procedures are really hard to test reliably. And they’re hard to code and debug. So you’re better off leaving your database to _store data_. Placing logic into component or services lets you test the logic without having to spin up databases with test data – using mocks or fakes or doubles lets you abstract away where the data is stored and test the logic of your apps. And that makes DevOps a lot easier since you can test a lot more during the build phase. And the earlier you find issues (builds are “earlier” than releases) the lest it costs to fix them and the easier it is to fix them.

### Managing Schema

What about schema? Even if you don’t have logic in your database in the form of stored procedures, you’re bound to change the schema at some stage. Don’t do it using manual scripts. Start using SQL Server Data Tools (SSDT) for managing your schema. Would you change the code directly on a webserver to implement new features? Of course not – you want to have source control and testing etc. So why don’t we treat databases the same way? Most teams seem happy to “just fix it on the server” and hope they can somehow replicate changes made on the DEV databases to QA and PROD. If that’s you – stop it! Get your schema into an SSDT project and turn off DDL-write permissions so that they only way to change a database schema is to change the project, commit and let the pipeline make the change.

The advantage of this approach is that you get a full history (in source control) of changes made to your schema. Also, sqlpackage calculates the diff at deployment time between your model and the target database and only updates what it needs to to make the database match the model. Idempotent and completely uncaring as to the start state of the database. Which means hassle free deployments.

## Automated Testing

I’ve already touched on this topic – using interfaces and inversion of control makes your code testable, since you can easily mock out external dependencies. Each time you have code that interacts with an external system (be it a database or a web API) you should abstract it as an interface. Not only does this uncouple your development pace from the pace of the external system, it allows you to much more easily test your application by mocking/stubbing/faking the dependency. Teams that have well-architected code are more likely to test their code since the code is easier to test! And tested code produces fewer defects, which means more time delivering features rather than fighting fires. Once again, good architecture is going to ease your DevOps!

Once you’ve invested in unit testing, you’ll want to start doing some integration testing. This requires code to be deployed to environments so that it can actually hit the externals systems. If everything is a huge monolithic app, then as tests fail you won’t know why they failed. Smaller components will let you more easily isolate where issues occur, leading to faster mean time to detection (MTTD). And if you set up so that you can deploy components independently (since they’re loosely coupled, right!) then you can recover quickly, leading to faster mean time to recovery (MTTR).

You’ll want to have integration tests that operate “headlessly”. Prefer API calls and HTTP requests over UI tests since UI tests are notoriously hard to create correctly and tend to be fragile. However, if you do get to UI tests, then good architecture can make a big difference here too. Naming controls uniquely means UI test frameworks can find them more easily (and faster) so that UI testing is faster and more reliable. The point surfaces again that DevOps is making you think about how you structure even your UI!

## Monitoring

Unfortunately, very few teams that I come across have really good monitoring in place. This is often the “forgotten half-breed” of the DevOps world – most teams get source code right, test right and deploy right – and then wash their hands. “Prod isn’t my responsibility – I’m a dev!” is a common culture. However, good monitoring means that you’re able to more rapidly diagnose issues, which is going to save you time and effort and keep you delivering value (debugging is not delivering value). So you’ll need to think about how to monitor your code, which is going to impact on your architecture.

Logging is just monitoring 1.0. What about utilization? How do you monitor how much resources your code is consuming? And how do you know when to spin up more resources for peak loads? Can you even do that – or do your web services require affinity? Ensuring that your code can run on 1 or 100 servers will make scaling a lot easier.

But beyond logging and performance monitoring, there’s a virtually untapped wealth of what I call “business monitoring” that very few (if any) teams seem to take advantage of. If you’re developing an e-commerce app, how can you monitor what product lines are selling well? And can you correlate user profiles to spending habits? The data is all there – if you can tap into it. Application Insights, coupled with analytics and PowerBI can empower a new level of insight that your business didn’t even know existed. DevOps (which included monitoring) will drive you to architect good “business monitoring” into your apps.

## Build and Release Responsibilities

One more nugget that’s been invaluable for successful pipelines: know what builds do and what releases do. Builds should take source code (and packages) as inputs, run quality checks such as code analysis and unit testing, and produce packaged binaries as outputs. These packages should be “environment agnostic” – that is, they should not need to know about environments or be tied to environments. Similarly your builds should not need connections strings or anything like that since the testing that occurs during a build should be unit tests that are fast and have no external dependencies.

This means that you’ll have to have packages that have “holes” in them where environment values can later be injected. Or you may decide to use environment variables altogether and have no configuration files. However you do it, architecting configuration correctly (and fit for purpose, since there can be many correct ways) will make deployment far easier.

Releases need to know about environments. After all, they’re going to be deploying to the environments, or perhaps even spinning them up and configuring them. This is where your integration and functional tests should be running, since some infrastructure is required.

## Conclusion

Good architecture makes good DevOps a lot easier to implement – and good DevOps feeds back into improving the architecture of your application as well as your processes. The latest trend of “shift left” means you need to be thinking about more than just solving a problem in code – you need to be thinking beyond just coding. Think about how the code you’re writing is going to be tested. And how it’s going to be configured on different environments. And how it’s going to be deployed. And how you’re going to spin up the infrastructure you need to run it. And how you’re going to monitor it.

The benefits, however, of this “early effort” will pay off many times over in the long run. You’ll be faster, leaner and meaner than ever. Happy DevOps architecting!

