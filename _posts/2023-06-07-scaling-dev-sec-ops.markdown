---
layout: post
title: 'Management by Exception - the Key to Scaling DevSecOps'
date: '2023-06-06 01:22:01'
image: /assets/images/2023/06/management/chili.jpg
description: >
  Tooling is an important aspect of DevSecOps - but culture and management style are key factors that can dramatically influence how you scale. In this post I'll discuss some thoughts about management by exception.
tags:
- devops
- security
---

1. TOC
{:toc}

> Image by [Pickled Stardust](https://unsplash.com/@pickledstardust?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/photos/4xc6i5BKPWs?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
  
I work for GitHub - so naturally I have a lot of conversations about tooling and products. However, let's take a step back and remember Donovan Brown's seminal definition of DevOps:

> DevOps is the union of people, process and products to enable continuous delivery of value to our end users.

You've also probably heard Peter Drucker's quote:

> Culture eats strategy for breakfast.

_Culture_ is the _people and product_ part of the DevOps equation, and are arguably more important than the _product_ or platform your teams are working with.

That's all well and good in a theoretical, high-level way. But how do we apply these principles in practice?

## Team Autonomy vs Enterprise Alignment

Many years ago, I heard Aaron Bjork and Buck Hodges from the Azure DevOps team talk about how Microsoft transformed their teams from a 2-year delivery cycle to a 3-week delivery cycle. This [excellent video](https://www.youtube.com/watch?v=WhRRGUmwoq4&t=10s) by my late friend and colleague Able Wang talks about this transformation and I highly recommend it.

One concept has always stood out to me when Microsoft spoke about this transformation: _team autonomy_ vs _enterprise alignment_. You can imagine these as two ends of a spectrum, with total team autonomy on one side and complete enterprise alignment on the other side.

To visualize these extreme ends of the spectrum, picture 300 rowboats vs the Titanic:

- the 300 rowboats can each turn very quickly
- each rowboat can travel fast or slow, according to how well the rowers gel together
- getting all 300 rowboats pointed in the same direction is a challenge
- communicating to all 300 rowboats is a challenge
- the Titanic only has a single direction
- the Titanic takes a long time to change direction
- communication on the Titanic is easier

Most organizations fall somewhere on the spectrum between team autonomy and enterprise alignment, and various points along the spectrum have advantages and disadvantages.

### Team Autonomy

Team autonomy means that teams are able to make decisions without filling in forms and logging tickets. To make this practical for software development, it means allowing teams to decide which programming languages and stacks they want to work with, what IDEs they want to use, and how they will build, test, scan, deploy and monitor their apps.

### Enterprise Alignment

Enterprise alignment is the vision and goal of the company and how that is worked out day-to-day. It defines how individuals and teams communicate, what their standards are, and what future direction is. It also defines the "non-negotiables".

In practice, successful organizations have a small, well-defined "core" of Enterprise Alignment, and then allow teams to have a large level of autonomy. Enterprise alignment defines the _what_ and let's teams define the minutia of the _how_.

## Tying in to DevSecOps

How does this tie into DevSecOps? Many organizations I work with have a centralized, command and control model. In other words, they lie much closer to the Enterprise Alignment side of the spectrum. Let's look at two examples: builds and security. We'll analyze each on both extremes: enterprise alignment and team autonomy.

## Considering Builds

### Extreme Enterprise Alignment

Many organizations have a "DevOps team". I really despise this language, since it makes DevOps the responsibility of some other team - after all, if I'm not on the DevOps team, then why should I care about DevOps? I think what most organization mean is that they have a team that is responsible for build and deployment automation.

The idea behind this team is to enable developers to code, and not have to worry about how to package, test, scan and deploy their apps. This leads to app developers not caring about operational issues, not building sufficient telemetry into their apps, not caring about security or scale or performance. After all, that all falls onto the "DevOps" team.

The supposed value-add is that there is a standardized build, test and deploy process, controlled by the DevOps team.

### Extreme Team Autonomy

When there is no enterprise alignment, it can look and feel like the wild west. Teams are deploying whenever and however they want, there is little or no code sharing and there are a plethora of tools since each team is using its own preferred tools and stacks.

While this allows agility in the "local" this ends up being a blocker at the "global" level. Teams optimizing for themselves end up being blocked by other teams (or blocking other teams) since there is no set contract for sharing code or apps and no set way to communicate.

### Well Balanced

A more balanced approach would be to have a small, well defined set of goals at the enterprise level that can guide teams and set a few non-negotiables. Thereafter, teams should be free to innovate within those boundaries.

How would we do this with builds? One way would be to standardize on a single build platform (say, GitHub) and then require teams to test, secure and monitor _their own apps_. This can be achieved by setting up branch policies to ensure that teams place these gates into their processes and making developers responsible for run-time operations of their apps. How teams test can be left up to them, as long as they test. If teams don't want to add telemetry, they are going to have a hard time running apps in production - so they will likely end up adding telemetry to make operations easier.

## Considering AppSec

We can also consider AppSec on the spectrum.

### Extreme Enterprise Alignment

Most organizations I work with have a Cyber security team. These teams are typically involved late in the development lifecycle and are the official gate-keepers to "going to prod". The idea is that this centralized team is the enterprise alignment for securing applications.

There are many problems with this extreme - poor developer experience, slowing release cycles and friction. When you add that security engineering skills are rare (1 security pro for every 800 developers is the current industry measure) then you get the additional problem that this does not scale.

The value-add for this would be a central place where security and risk are surfaced and managed. Unfortunately, the bottleneck and friction this model creates negates any benefits.

### Extreme Team Autonomy

On the other extreme, teams are not bound to any security standards at all, leading to risk for the company. If teams are scanning their code, dependencies and secrets, they're using disparate tools and processes and it is nearly impossible to manage risk at scale.

### Well Balanced

How can we balance these requirements - centralized risk management and good developer experience?