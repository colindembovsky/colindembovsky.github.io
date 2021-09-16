---
layout: post
title: VSTS, One Team Project and Inverse Conway Maneuver
date: '2018-05-03 20:26:35'
tags:
- devops
---

There are a lot of ALM MVPs that advocate the "One Team Project to Rule Them All" when it comes to Visual Studio Team Services (VSTS) and Team Foundation Server (TFS). I've been recommending it for a long time to any customer I work with. My recommendation was based mostly on experience - I've experienced far too much pain when organizations have multiple Team Projects, or even worse, multiple Team Project Collections.

While on a flight to New Jersey I watched a fantastic talk by Allan Kelley titled [Continuous Delivery and Conway's Law](https://www.youtube.com/watch?v=Cu0AU8vw3xw). I've heard about [Conway's Law](http://www.melconway.com/Home/Conways_Law.html) before and know that it is applied to systems design. A corollary to Conway's Law, referred to as [Inverse Conway Maneuver](https://www.thoughtworks.com/radar/techniques/inverse-conway-maneuver), is to structure your organization intentionally to promote a desired system architecture. This has a lot of appeal to me with regards to DevOps - since DevOps is not a tool or a product, but a culture: a way of thinking.

With these thoughts in mind, as I was watching Kelley's talk I had an epiphany: you can perform an Inverse Conway Maneuver by the way you structure your VSTS account or TFS install!

## What is Conway's Law?

In April 1968, Mel Conway published a paper called "How Do Committees Invent?" The central thesis of this paper is this: "Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure." In other words, the design of your &nbsp;organizational and team structures will impose itself on your system designs. At least, if they are out of sync, you will experience friction. The Inverse Conway Maneuver recognizes Conway's Law and makes it intentional: use organizational and team structure to _promote desired systems design_. For example, distributed teams tend to develop more modular products, while centralized teams tend to develop monoliths.

Historical side-note: Conway's paper was rejected by Harvard Business Review in 1967 since Conway had "failed to prove his thesis". Ironically, a team of MIT and Harvard Business School researchers published a paper in 2015 which found "strong evidence" to support the hypothesis.

How does this apply to VSTS and TFS? I'll explain, but it's awkward having to type "VSTS and TFS". For the remainder of this blog I'll just write VSTS - but the same principles apply to TFS: the VSTS account is like a TFS Team Project Collection. If we equate VSTS account to Team Project Collection, then the rest of the hierarchy (Team Project, Team etc.) is exactly equivalent. In short, when I say VSTS account I also mean TFS Team Project Collection.

## One Objective: Deliver Value to End Users

In days gone by, IT was a service center to Business. Today, most organizations are IT companies - irrespective of the industry they operate in. Successful businesses are those that embrace the idea that IT is a business enabler and differentiator, not just a cost center. There should be very little (if any) division between "business" and "IT" - there is one team with one goal: deliver value to customers. Interestingly the definition of DevOps, [according](http://donovanbrown.com/post/what-is-devops) to [Donovan Brown](https://twitter.com/DonovanBrown) (Principal DevOps Manager at Microsoft), is "the union of people, process and products to enable continuous **delivery** of **value** to our **end**  **users**" (emphases mine).

One Objective means everyone is aligned to the overall goal of the business. If you look at two of the [Principles behind the Agile Manifesto](http://agilemanifesto.org/principles.html):

- Business people and developers must work together daily throughout the project.
- The best architectures, requirements, and designs emerge from self-organizing teams.

you'll see a common theme: aligning everyone to the One Objective. The point I'm making is that there needs to be a "one team" culture that permeates the organization. DevOps is _cultural_ before it's about tools and products. But putting the thinking into practice is no easy task. Fortunately, having the correct VSTS structures supports an Inverse Conway Maneuver.

## One Team Project

So how do you use VSTS for an Inverse Conway Maneuver? _ **You have a single VSTS account with a single Team Project** _.

Having all the work in a single Team Project allows you to view work at a "portfolio" (or organizational) level - that is, across the entire organization. This is (currently) impossible to do with multiple VSTS accounts and very difficult with multiple Team Project Collections. Even viewing portfolio level information with &nbsp;multiple Team Projects can be difficult. Work item queries are scoped to Team Projects by default; widgets, dashboards, builds, releases, package feeds, test plans - these all live at Team Project level. If you have multiple Team Projects you've probably experienced a fragmented view of work across the organization. Interestingly, that's probably not only from a VSTS point of view, but this structure (by Conway's Law) is probably responsible for silos within the organization.

Hence the recommendation for a single "Team Project to Rule Them All." Not only will this allow anyone to see work at a portfolio level, but this allows teams to share source repositories, build definitions, release definitions, reports and package feeds. It's a technical structure that encourages the One Objective.

## Teams

I can hear you already: "But I have 500 developers/analysts/testers/DBAs/Ops managers (let's say engineers, shall we?) - how can I possibly organize them under a single team project?" That's where Teams come in. Teams allow organizations to organize work into manageable sets. When you're an engineer and you want to deliver value, you probably only need a general idea of the One Objective, rather than having to know the minutia of every bit of work across the entire organization. Having your team's work in a separate ring-fenced area allows you to focus on what you need day-to-day. You can go up to the portfolio level when you need a wider context - but you probably don't need that every day. Leadership will more likely spend most of their time looking at work at the portfolio level rather than all the way down to the minutia of the team-level work.

So how should you organize your teams? Again, Conway's Law is going to have enormous impact here. Do you have a 3-tier application? Then you might be tempted to create a DBA Team, a Service Team and a UI Team. Perhaps create a Mobile team and a Data Analytics Team too. Surely that's reasonable, right?

The answer, to quote Consultese (the dialect of the consultant) is: _It Depends_. Perhaps that is the way to go since that is how your application is architected. But that could be boxing you in: horizontally composed teams violate the Agile principle of cross-functional teams. A better approach is to have your teams composed around functional area or module. Where possible, they should be loosely coupled. Again by Conway's Law this will start reflecting in your app architecture - and you'll start seeing your applications become loosely coupled services. Have you ever wondered why micro-services are so popular today? Could it be the Agile movement started to break huge monolithic organizations into small, loosely-coupled, cross-functional and self-organizing teams, and now we're starting to see that reflected in our architecture? Inverse Conway Maneuvers at work.

In short, create Teams around functional areas and use Area Paths to denote ownership of that work. If an Epic/Feature/Story belongs to a team, put it in the Area Path for that team and it appears on their backlogs. Another tip is that your Area Paths should be durable (long-lived) while your work items should not: work items should have a definite start and end date. Don't make an Epic for "Security" since that's not likely to end at a specific date. Rather, have an Area Path for Security and place work items in that area path.

## Organizational Dimensions in VSTS

There are four axes that most organizations use to organize work: functional area, iteration, release and team. Unfortunately, VSTS only really gives us two: Area and Iteration. While Release Management in VSTS is brilliant, there isn't yet a first-class citizen for the concept of a Release. And while you can create a custom Team Field in TFS and slice teams on that field, you can't do so in VSTS, so you have to munge Team and Area Path together. In my experience it's best not to fight these limits: use Area Path to denote Team, use iterations to time-box, and if you really need a Release concept, add a custom field.

## Areas and Work Item States

Organizations will still need inter-team communication, but this should be happening far less frequently that intra-team communication. That's why we optimize for intra-team communication. It's also why co-locating a team wherever possible is so important. If you do this, then by Conway's Law you are more likely to end up with modules that are stable, resilient, independent and optimized.

We've already established that vertical Teams are tied to Area Paths. Each Team "owns" a root area path, typically with the same name as the Team. This is the area path for the team's backlog. The team can then create sub-areas if they need do (leave this up to the team - they're self-organizing after all). Kanban boards can be customized at the team level, so each team can decide on whatever columns and swim-lanes they want in order to optimize their day-to-day work. Again, leave this up to the team rather than dictating from the organizational level.

Work Item states can't be customized at Team level - only at the Team Project level. If you only have a single Team Project, that means every team inherits the same work item states. This is actually a good thing: a good design paradigm is to have standard communication protocols, and to have services have good contracts or interfaces, without dictating what the internals of the service should look like. This is reflected by the common "language" of work item state, but let's teams decide how to manage work internally via customized Kanban boards. Let Conway's Law work for you!

## Iterations

While teams should have independent backlogs and areas, they should synchronize on _cadence_. That is, it's best to share iterations. This means that teams are independent during a sprint, but co-ordinate at the end of the sprint. This enforces the loose coupling: teams are going to have dependencies and you still want teams to communicate - you just want to streamline that communication. Sharing iterations and synchronizing on that heartbeat is good for the Teams as well as the software they're delivering.

## Enterprise Alignment vs Team Autonomy

The VSTS team have a single Team Project Collection for their work. They speak about Enterprise Alignment vs Team Autonomy. I heard a great illustration the other day: the Enterprise is like a tanker - it takes a while to turn. Agile Teams are like canoes - they can turn easily. However, try to get 400 canoes pointed in the same direction! As you work to self-organizing teams, keep them on the One Objective so that they're pointed in the same direction. Again, that's why I like the One Team Project concept: the Team Project is the One Direction, while Teams still get autonomy in their Team Areas for daily work.

## Organizing Source Code, Builds, Releases, Test Plans and Feeds

If you have a single Team Project, then you'll have a challenge: all repositories, builds, releases, test plans and package feeds are in a single place. Builds have the concept of Build Folders, so you can organize builds by folders if you need to. However, repos, releases, test plans and feeds don't have folders. That means you'll need a good naming strategy and make use of Favorites to manage the noise. In my opinion this is a small price to pay for the benefits of One Team Project.

## Security

Often I come across organizations that want to set up restrictions on who can see what. In general: don't do this! Why do you care if Team A can see Team B's backlog? In fact it should be encouraged! Find out what other teams are working on so that you can better manage dependencies and eliminate double work. Same principle with Source Code: why do you care if Team C and see Team D's repos?

There are of course exceptions: if you have external contractors, you may want to restrict visibility for them. In VSTS, deny overrides allow, so in general, leave permissions as "Not Set" and then explicitly Deny groups when you need to. The Deny should be the exception rather than the rule - if not, you're probably doing something wrong.

Of course you want to make sure you Branch Policies (with Pull Requests) in your source code and approval gates in your Releases. This ensures that the teams are aware of code changes and code deployments. Don't source control secrets - store them in Release Management or Azure Key Vault. And manage by exception: every action in VSTS is logged in the Activity Log, so you can always work out who did what after the fact. Trust your teams!

## Conclusion

Don't fight Conway's Law: make it work for you! Slim down to a single VSTS Account with a single Team Project and move all your Teams into that single Team Project. Give Teams the ability to customize their sub-areas, backlogs, boards and so on: this gives a good balance of Enterprise Alignment and Team Autonomy.

Here is a brief summary of how you should structure your VSTS account:

- A single VSTS Account (or TFS Team Project Collection)
- A single Team Project
- Multiple Teams, all owning their root Area Path
- Shared Iteration Paths
- Use naming conventions/favorites for Repos, Releases, Test Plans and Feeds
- Use folders for organizing Build Definitions
- Enforce Branch Policies in your Repos and use Approval Gates in Release Management
- Have simple permissions with minimal DENY (prefer NOT SET and ALLOW)

Happy delivering!

