---
layout: post
title: 'Spicy Takes 🌶️🌶️🌶️ on RSA 2023'
date: '2023-05-01 01:22:01'
image: /assets/images/2023/05/rsa/chili.jpg
description: >
  I was recently at RSA for the first time. I have some spicy takes from the week.
tags:
- security
---

1. TOC
{:toc}

> Image by [Pickled Stardust](https://unsplash.com/@pickledstardust?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/photos/4xc6i5BKPWs?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
  
I was at RSA last week in San Francisco. The highlight of the week was a talk by [Shannon Lietz](https://www.linkedin.com/in/shannonlietz/), who I met briefly at GitHub HQ during the week. More on this later.

I visited expo area and I had great conversations with GitHub customers as well as GitHub technology and services partners. I was looking for overall trends and trying to get a pulse on the industry - and coming from a developer background, the security world is both fascinating and foreign to me!

# Spicy Takes

There are a couple of key themes that I took away from the week, and I present them here in order of spiciness:

1. 🌶️ Culture eats application security for breakfast
1. 🌶️🌶️ Organizations that don't invest in developers are not serious about security
1. 🌶️🌶️🌶️ Security tools are a dime a dozen

## 🌶️ Culture eats application security for breakfast

I am used to the phrase "culture eats tooling for breakfast" in the context of DevOps. You can have the most amazing tools, but if you have a dysfunctional culture, _tools will not help you succeed_. Many of the conversations I had this week presented echoes of this sentiment, but in the context of security. So it is easy to turn the phrase into _culture eats application security for breakfast_.

But what does this really mean? I was struck by how little emphasis was placed on culture as a foundation and pillar for application security. A culture that isolates and separates developers and security professionals will struggle to be effective at AppSec.

[Conway's Law]({% post_url 2018-05-03-vsts-one-team-project-and-inverse-conway-maneuver %}) teaches us that the communication structures of organizations is invariably reflected in the application architectures of those organizations. It's no surprise when we look at the popularity of n-tier applications in the late 90's and early 2000's - these mirror the top-down, hierarchical management structures that were prevalent in those times. As Agile gained popularity and management changed to smaller, more autonomous teams, we saw the proliferation of microservices.

This is why we must consider the impact of how our developers and security teams communicate and collaborate if we want to succeed at AppSec. We cannot get away from Conway's Law. If we continue to bolt security teams onto developer teams late in the development life cycle as a mess of bureaucratic red tape, then AppSec will be continue to fail.

You've heard the mantra "shift left", and today no self-respecting security pro worth their salt won't talk about this concept. But simply deploying another tool in an automated build has limited efficacy - we must "shift the culture left".

Teams with good tools and bad culture are less effective than teams with good culture and bad tools. Ultimately, we need to progress to teams that have both good culture _and_ good tools.

## 🌶️🌶️ Developers, developers, developers!

Following on from the culture discussion above, we have to pivot to the key to effective AppSec: _the developer_. Changing culture is going to require renewed investment in developers as well as a shift in the roles and responsibilities of security professionals.

One highlight of the week was the DevOps Connect talk by Shannon Lietz. I particularly remember her saying, "To be effective in security, we must _translate security into developer_."

I realize that I was at a _security_ conference, but I realized that there are very few companies today that are looking to solve security by investing in developers. And I will go even further: companies that do not look to solve AppSec by investing in developers are doomed to fail at AppSec.

AppSec is a fascinating intersection between developers and security professionals. These two groups typically speak different languages and have different lenses through which they view the world. This is why I resonated with Shannon's statement - companies that fail to translate security into language, processes and tooling that developers understand are not serious about security. And as part of that, they must transform how security professionals work too!

### Security professionals as Enabling Teams

[Team Topologies](https://teamtopologies.com/) does a great job in creating language around how to design teams within an organization. This is another area that suffers a severe lack of investment - companies don't typically think about how they design their teams or how their teams communicate. Without going into the four types of Teams, at a high level, developers should be Stream Aligned Teams and security professionals should become Enabling Teams.

In short, the security teams should work on _enabling developers_ to write secure code, fix vulnerabilities and become the first line for security. If your security professionals are doing all the security work, they will always be a bottleneck. Organizations can scale AppSec and scarce security skills by taking this approach. This is what I think true "shift culture left" means in the context of AppSec.

## 🌶️🌶️🌶️ Security tools are a dime a dozen

Most of the vendors at the expo seemed cookie-cutter, using oft-repeated catch phrases (like the ubiquitous "shift left" and "go faster") but didn't seem to bring anything new or fresh to AppSec.

There are some critical dimensions that companies must consider when evaluating and rolling out security tools:

1. Developer productivity
1. Reduced friction
1. Visibility
1. Scalability

I was disappointed to see that very few tools in the AppSec space addressed these dimensions. Slapping another tool into the mix isn't going to be effective - you must address these dimensions.

### Developer productivity

Moving fast isn't just about new features: your _security response_ velocity will never be faster than your _developer velocity_. It's simple to illustrate this point: let's say that your commit-to-production lead time is 3 days; in that case it stands to reason that your time to remediate cannot be _faster_ than 3 days. Speed is a critical component of staying secure.

### Reduced friction

Another great quote from Shannon Lietz is: "Developers don't talk about security tools unless they _make security folks go away_." When I was a developer, security were the people that blocked your deployments. Security tools only slowed me down. It wasn't until I saw a _developer-focused_ security tool that I realized that security doesn't have to be a blocker! Shannon's sentiment is spot-on.

Developers are smart - and hate process when it adds no value to what they do. They tend to find workarounds for any process that introduces more friction. Therefore, any tool that adds friction is doomed to fail. Tools must reduce friction for developers to be successful.

### Visibility

One of my customers has a security tool that performs formal method analysis. They use this tool heavily - but they cannot struggle to collate results and see status over multiple projects. They can switch to the tool UI, but this adds to friction. The lack of visibility in the developer workflow is limiting the effectiveness of this tool.

Another part of visibility is _metrics_. Most teams will talk about Mean Time to Detect (MTTD) and Mean Time to Repair (MTTR), but do not define these or track them effectively. There doesn't seem to be a consensus on what AppSec metrics are the most important or how to track them.

### Scalability

The industry standard ratio for security professionals to developers is 1:800. This is why shifting your security professionals to enabling teams (see above) is so critical - it is the only way to scale AppSec effectively. But you will struggle to do this if your security tools cannot support this shift.

## GitHub Advanced Security

I often ask the question, "Why do you think GitHub got into AppSec at all?" The answer is fairly simple: even though security tools and practices have been around for two decades, AppSec is still failing. And the major reason is that it _is not developer centric_. GitHub is uniquely positioned to bring security to developers in a way that reduces friction, empowers developers and scales AppSec teams. Since it is the heart of the developer workflow, this is a powerful way to really shift both tooling and culture left.

# Conclusion

We still have a lot of work to do. AppSec in the industry isn't as successful as it should be, and organizations must consider both tools and culture in combination in order to improve. Organizations must invest in developers, shift security pros to enabling teams and ensure that they deploy tools that support these shifts instead of hinder them. I was again reminded of how fortunate I am to be at GitHub, where we are moving AppSec forward.

Happy securing!