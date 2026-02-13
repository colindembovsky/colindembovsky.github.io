---
layout: post
title: DevOps Benefits of Limiting WIP
date: '2020-11-19 01:57:40'
description: >
  Generally limiting WIP is discussed in the context of work item tracking - but too much WIP has detrimental effects on branching, testing, architecture and technical debt too!
tags:
- devops
- process
---

1. TOC
{:toc}

We've recently completed several DevOps Assessments for various customers. These assessments go through work item tracking and planning, source control practices, automated build and test, release and test management, database DevOps, security, architecture, ops and monitoring and team structures. It's a lot to cover - and I've noticed a common theme that comes up over and over: almost every team has too much work in progress (WIP).

This generally comes up early on when we talk about work item tracking, which I won't cover here since there are plenty of posts (including some of mine) that cover why you should limit WIP for work items. But there are numerous impacts of having too much WIP in other areas of the delivery lifecycle. In this post I'll go through some of them.

## Branching

I'm a fan of [GitHub Flow](https://guides.github.com/introduction/flow/) (also known as trunk development or master development).

> Note: GitHub Flow is not the same as GitFlow. I don't like GitFlow for several reasons, but the most glaring is that it is at heart a manner of code-promotion: I think that binary promotion is far better.

At a high level, the idea behind GitHub Flow is that you have a single stable branch (master) and every User Story is developed on a _topic_ branch (sometimes called a _feature_ branch too, but the word feature can get overloaded). Once you're code complete, you create a Pull Request (PR) to merge the isolated topic-branch code into the master branch.

This lends itself well to Continuous Integration and Continuous Delivery (CI/CD) since the code change is usually small. Several topic branches can be in flight at the same time.

However, if you have too many topic branches because you have too much WIP, then you quickly get into situations where topic branches cannot be merged because the large number of branches is overwhelming the test environments. Also, if you release in large batches, you'll probably want to merge topic branches together before merging to master to do integration testing.

I've seen teams introduce a second "semi-stable" branch between master and the topic branches (often called DEV or INT). The idea is that topic branches merge to the DEV branch for integration testing and the DEV branch is then merged to master for release.

However, this isn't solving the issue. Now the DEV branch becomes congested. To make matters worse, teams that use this approach often have product owners that want certain features to go to PROD while other untested features should not. Now developers are not merging early, they're deferring merging to some later date when the rest of the process is "ready" for the code. This leads to stale branches and an accumulation of merge debt and general mess.

In short, reducing the number of topic branches by limiting WIP reduces the number of isolated branches (and corresponding test environments) required to release completed code. Teams can then use GitHub Flow without needing interim branches and they won't accumulate merge debt.

Ultimately, limiting WIP is going to simplify your branching strategy!

## Testing

Teams that have high or uncontrolled WIP generally have huge resource contention on test environments. Ideally, each story in flight has its own branch in source control and its own environment. That way each story can be developed and tested in isolation. However, if you have do not have WIP limits, then you fast end up running out of test environments.

Spinning up environments dynamically can certainly help, but often this is not cost effective. Again teams are left trying to come up with workarounds. For example, sharing environments: which makes test results harder to decipher since if there are multiple builds in an environment at the same time and there are failures, which build is responsible? How do you handle conflicting changes and dependencies?

I hear the astute reader ask, "What about integration testing? How do you test the combined effects of multiple stories that are in flight together?" Well, if there are a low number of these, you can easily merge one story into another (or create a combined branch) for this sort of testing. Of course, if you have lower WIP, then your cycle times will reduce proportionately (via Little's Law, since lowering WIP also lowers cycle times) so you can release one story at a time, meaning you have less time where there are multiple incomplete stories at any one time.

Simply put, reducing WIP will increase your ability to test and reduce test environment management overhead.

## Architecture

Rather than stating how reducing WIP can affect architecture, this one works that other way around. Monolithic architectures, by nature of being large, tend to require batching changes and releases, which drives up the WIP. However, decomposing systems into loosely coupled services that can be released independently (ahem, microservices, anyone?) means that changes to components can be released far quicker. And, again via Little's Law, if we reduce cycle time, we'll be reducing WIP.

In other words, modular, testable, loosely-coupled components allow teams to reduce WIP.

## Technical Debt

Every application has some form of technical debt. And most product owners don't care about fixing technical debt, particularly because it is often abstract: it's not a new feature or a bug-fix. However, technical debt, like unpaid mortgage debt, tends to get worse the longer you don't pay it down.

Dealing with technical debt will ultimately make systems (and teams) more agile, since there can be more focus on new features and updates, rather than bug-fixing. However, even if your team recognizes the value of paying down technical debt, if there is no elasticity in the backlog or project plan because there is too much WIP, then your team will never have time to pay down the technical debt!

Lowering WIP allows teams time to actually address technical debt. And as teams pay down the technical debt, they become faster at new features and at bug-fixes, which ultimately is going to make users happier.

## Conclusion

As you can see, lowering WIP not only has an impact on the project plan and work item tracking, but can simplify branching strategies, lower test overhead and resource contention, aid modernizing architecture and reduce technical debt. Whatever you're doing to improve your DevOps, make sure you're relentless about lowering your WIP!

Happy delivering!

