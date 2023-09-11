---
layout: post
title: 'Measuring the impact of Developer Experience and GitHub Copilot'
date: '2023-07-11 01:22:01'
image: /assets/images/2023/07/measuring/productivity.jpg
description: >
  Measuring the impact of Developer Experience and GitHub Copilot is a complex subject. Understanding leading and lagging indicators can help organizations measure the right things, and thus prove out the value of good Developer Experience in general, as well as the impact of GitHub Copilot.
tags:
- ai
- development
---

1. TOC
{:toc}

> Photo by <a href="https://unsplash.com/@schmaendels?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Andreas Klassen</a> on <a href="https://unsplash.com/photos/gZB-i-dA6ns?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

GitHub Copilot is radically transforming the software industry and highlighting the importance of Developer Experience (DevEx) as a key enabler to business success.

GitHub has published studies showing that developers are [55% faster with GitHub Copilot than without](https://github.blog/2022-09-07-research-quantifying-github-copilots-impact-on-developer-productivity-and-happiness/). Customers using GitHub Copilot are reporting numbers inline with those studies: Mercado Libre reports a [50% reduction in time spent writing code](https://github.com/customer-stories/mercado-libre), and Duolingo is seeing a [25% increase in developer speed](https://github.com/customer-stories/duolingo).

Accurately measuring the return on investment (ROI) for DevEx in dollar terms is nuanced, difficult and complex. For Copilot, it is more difficult. This is despite the fact that there have been many attempts to measure productivity - from counting lines of code to logging hours that developers spend in their IDEs to measuring velocity. Many of these methods are insufficient or subject to gamification.

GitHub Copilot is really a _productivity tool_. Productivity is so inextricably intertwined with DevEx, they can be spoken of synonymously. Any attempt to measure the value of GitHub Copilot must be tied to measuring DevEx in general.

> [DORA Metrics](https://dorametrics.org/) have long been used to measure DevOps: lead time, deployment frequency, mean time to recovery and change failure rate. When coupled with flow metrics as defined by Daniel S. Vicanti in [Actionable Agile Metrics for Predictability](https://actionableagile.com/resources/publications/aamfp/) - cycle time, work in progress, throughput and work item age - organizations have a powerful set of metrics that can track how well they produce software. The [SPACE framework](https://queue.acm.org/detail.cfm?id=3454124) is an excellent framework for understanding developer productivity.

Why is it so hard to measure developer productivity? Firstly, it's hard to define DevEx. There are many different opinions about what developer productivity is. Furthermore, both _perceptual_ (qualitative) as well as _workflow_ (quantitative) metrics should be considered. Measuring developer satisfaction is just as important as measuring how fast they work: happy developers are productive developers, since they spend more time coding and shipping great products, and are more likely to stay with your company. DevEx is multi-dimensional, so no single metric is going to tell the whole story.

> You can read a much more detailed analysis of perceptual and workflow metrics and the dimensions of DevEx in [this paper](https://queue.acm.org/detail.cfm?id=3595878).

To fully understand how to measure developer productivity, we have to understand how _leading_ and _lagging_ indicators work. Let's unpack these concepts.

## Leading and lagging indicators

Leading indicators are measures of _inputs_ into a system. They help us to predict how the system will perform _in the future_. Typically, these are fairly easy to measure and can be influenced in a short period.

A good example of a leading indicator for a development team is the count of work items on the backlog, or committed to a sprint. This is easy to measure (just check the backlog) and easy to influence - we can immediately remove (or add) items committed to a sprint.

Lagging indicators are measures of the _outputs_ of a system. They help us understand how something happened in the system _in the past_: they are retrospective in nature. Typically, these require longer time periods to measure. Lagging indicators are also the result of aggregated leading indicators, so you can't directly affect them. 

A good example of a lagging indicator for a development team is how many items are delivered in a sprint. Measuring this requires us to wait until the end of the sprint, so it takes a while to measure. This count can’t be _directly_ changed - you can try to add more committed items in the next sprint, but that may result in more bottlenecks or contention for testing environments or any number of other issues that don’t actually increase the number of items completed.

## Applying indicators to developer productivity

Let's apply these concepts to the problem of measuring developer productivity. Remember _developer productivity isn't an end in itself_ - it's a means to an end. To what end? Ultimately, it's to make our business successful! A business may have productive teams and not do well in the market. So what are we trying to achieve? And how would we know that we've been successful?

We may want to ask questions like:

- How can we develop faster?
- How can we reduce risk?
- How can we improve quality?
- How can we innovate more?

But how would we measure those? How would we know we've been successful? We could measure some of these:

- Cycle times - how fast can we complete work?
- Frequency of deployments - how frequently can we deploy?
- Bugs - how many do we have in a release?
- Vulnerabilities - how many do we have in a release?
- How many code reviews do we do (and how fast do we do them)?
- How much burnout do we see?
- How easy is it to attract (and retain) talented people?
- How much are we innovating vs maintaining?

These metrics give you insight into how well your team is performing - but even these must be analyzed in the context of the business. Are you attracting and retaining more customers? How delighted are your customers with your products and services? How competitive are you in your market? Delivering faster won’t help the business if you’re delivering the wrong things.

Let's imagine that we measure the number of bugs in a release. Release A had 3 bugs, and Release B had 5 bugs. This tells us that there is a problem somewhere, since the number of bugs increased. But what? This is where we see the challenge of metrics - how do we interpret what happened? Perhaps we added a lot of code and didn't add enough tests. Perhaps our senior developers were too busy to do proper code reviews, so they missed some bad code. Perhaps a developer was burning out and just pushed code without taking care to test it properly. Multiple inputs may have affected an output that we're not happy with.

## Measuring the right things

What does this mean for measuring developer productivity and the value of GitHub Copilot? Measuring lines of code that Copilot produced or how many prompts were accepted are _leading indicators_ that should have an impact on lagging indicators down the line. In other words, _the immediate improvement_ (which is easier to measure) will result in affecting the _future impact_ (which is harder to measure). However, the dollar value impact (ROI), is typically tied to the _lagging indicators_.

What does that mean? Here's the critical concept: _measuring flow and other life cycle metrics is the best way to measure the dollar value of GitHub Copilot_. This is the challenge to organizations: to mature in tracking these metrics so that they can really see the impact of developer productivity on business outcomes.

There is a caveat here: GitHub Copilot is a tool meant primarily to increase individual productivity at the task level. While making developers faster at task completion will certainly impact team performance metrics like cycle times, task completion is not the only factor affecting team performance. For example, team performance involves synchronization (code review must be scheduled into the reviewer’s calendar), meetings, design sessions and many other processes and ceremonies.

## Assessing the value of GitHub Copilot

The hypothesis is that by utilizing GitHub Copilot we can affect leading indicators like speed of coding, quality of code, test coverage and speed of code review. Improving these indicators will affect the lagging indicators like velocity and deployment frequency, quality, mean time to resolution (MTTR) and risk.

Unsurprisingly, the lagging indicators are typical DevSecOps metrics! These typically require longer periods of time to measure. Furthermore, when they change, it's not always easy to analyze _why_ they changed.

If you look at the above list, you'll see that the leading indicators are fairly easy to affect, and don't require long time periods to measure. For a sprint (typically 2 - 4 weeks) we can easily measure how many items we delivered, or how many bugs we found or how long code reviews took. If we found few bugs and completed code reviews quickly, that should allow us to deploy more frequently. We can also improve these measures directly. For example, if we want to improve code review times, we can add automated quality gates that need to pass before code review. This can help ensure that code has higher quality by the time a reviewer opens it, leading to faster review times.

To tie this back to GitHub Copilot - if you really want to measure its impact on the team, you have to look beyond how many suggestions were accepted (a leading indicator) and measure lagging indicators. If you use GitHub Copilot, you should see improvements in the following areas:

- **More frequent deployments/reduced cycle times** Developers are spending less time hand-coding boilerplate code and searching for answers outside the IDE and so can complete tasks faster. GitHub Copilot is generating unit tests and documentation - all tedious, labor-intensive tasks that GitHub Copilot can do in milliseconds. This will lead to improved cycle times - and improved DevEx.
- **Fewer build failures** Developers can use Copilot Chat to explain code, meaning they can understand code more deeply. They can understand the impact of changes more clearly, and should lead to better code. As GitHub Copilot generates unit tests, buggy code is fixed before it's even pushed to the repository. Copilot Chat can help developers debug and fix problems as the code is being written. When coupled with branch protection rules, status checks, and custom deployment rules, this should all translate into fewer build failures.
- **Improved code quality and higher test coverage** GitHub Copilot can be used to generate test cases and test data faster, which should lead to more code coverage, which in turn will improve quality.
- **Faster code review times** Since GitHub Copilot is like having a second developer with you all the time, developers can generate good code, understand existing code, debug code and generate tests for code all before the code review. This means that by the time the code reaches review, it's higher quality, which should reduce the time needed to review it. Reviewers can use Copilot Chat to understand the impact of a proposed change by asking it to “explain this code”.
- **Fewer security vulnerabilities and improved MTTR** Copilot Chat is an excellent way to scale AppSec since it can guide developers in fixing security vulnerabilities without the need to involve a security professional. Furthermore, with AI filters on code suggestions, it is less likely to generate code suggestions with security vulnerabilities. This means that MTTR should improve and risk should be lowered. Recent research suggests developers intend to spend their new found time in code review and vulnerability remediation. 
- **Better flow metrics** Cycle times should be improved, and Work in Progress (WIP) should be lowered. When developers are faster at their tasks, they work on fewer things at the same time, reducing the overhead of context switching, allowing them to spend more time "in the zone" as well as reducing cognitive load. Furthermore, work item age should decrease (since work items will be completed faster). All of this works to improve throughput.
- **Accelerated developer growth** The Collaborative Software Process study shows that pair programming speeds development, improves quality and improves developer experience. GitHub Copilot allows every developer to have a pair programmer, even when remote. Furthermore, Copilot Chat acts like a "just in time" coach that can help developers grow their expertise. 
- **Better talent acquisition and retention** Happy developers are typically productive developers, but the corollary holds too: productive developers are typically happy developers. This has the dual benefit of attracting talent (developers love to work for high performing teams) as well as being good for the business, since developer churn costs in time and lost “tribal knowledge”. Furthermore, because of the improvements in quality and speed, developers will spend less time burning out, which is good for both talent acquisition and retention.

## The metrics challenge

The challenge with these metrics is that _they take time to measure_. And many organizations don't even have a baseline for some of these metrics. If organizations are going to be able to show the value of GitHub Copilot and improved DevEx, they are going to have to get to grips with these DevSecOps metrics, such as those from DORA, ActionableAgile and SPACE.

To further complicate things, many of these metrics have interdependencies. Optimizing one part of the development life cycle may highlight bottlenecks and inefficiencies in other parts of the development life cycle that could prevent the lagging indicators from improving. For example, let's say that you give your developers GitHub Copilot and they start coding faster and completing tasks faster. Now you have more code reviews than before - and you could end up overwhelming senior developers that perform the code reviews, and they become a bottleneck that prevents you from deploying more frequently. So we see that the lagging indicators are related to an aggregation of the leading indicators, and we must take this into account when doing any analysis.

You cannot get Copilot Chat to help you fix a vulnerability if you can't find the vulnerability, so you need good Application Security (AppSec) tools. You cannot attain more frequent deployments by improving developer speed alone - you have to invest in automation to build, test, scan, package and deploy your code. Improving cycle times won't help if you're not truly transforming the software delivery life cycle to be agile. And team performance improvements require streamlining processes and removing red tape, not just making individuals faster.

## Perceptual vs workflow metrics

Most of the above discussion has focused on _workflow_ (system) metrics. Even if the effect of these is understood, organizations must not forget the value of _perceptual_ metrics. These are informed by how developers _feel_ about GitHub Copilot and DevEx in general. Just as leading indicators interact with each other in complex ways to affect lagging indicators, perceptual metrics play an important role in DevEx. Any program to measure DevEx and the value of GitHub Copilot must include perceptual metrics such as how developers feel about the development process and their tools. More perceptual metrics are defined in this paper.

Perceptual metrics are best measured by surveys and self-assessments. They must be carefully designed to take into account bias and avoid survey fatigue. Organizations without expertise in these areas should consider outsourcing this kind of study to experienced partners.

Once the perceptual metrics have been obtained, organizations should analyze the perceptual and workflow metrics together with business key performance indicators (KPIs) in order to attain a clear, accurate picture of DevEx and value.

# Conclusion

When organizations look at the return on investment (ROI) for investing in DevEx (including deploying GitHub Copilot) multiple dimensions must be considered. Analyzing which metrics will be impacted by improvements is a complex activity with many nuances. Organizations should start to analyze both input (leading) and output (lagging) metrics so that they can develop a fuller understanding of how productive their developers are individually, as well as how productive teams are. Ultimately the goal of such measurement is to help improve productivity and DevEx to accelerate achieving business outcomes.

What can organizations do today to improve developer productivity? First, start by asking developers what their view of DevEx and productivity is. Then start measuring both input and output metrics as defined above with a view to discovering where to most effectively invest to improve.
