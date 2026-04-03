---
layout: post
title: 'From Sprints to Swarms, Part 1: AI Made Code Cheap But Delivery Hard.'
date: '2026-03-30 09:00:00'
image: /assets/images/2026/03/carrigae.png
description: >
  In the first part of this series, we examine how AI made code generation cheap while shifting bottlenecks to delivery. Pull requests are the critical unit of flow, exposing pressure on review, testing, merge speed, and human-agent coordination. We challenge current processes and assumptions and probe into what teams need to think about as they embrace agentic engineering.
tags:
- ai
- devops
- process
---

1. TOC
{:toc}

## Series Position

You've heard all the hype: "The age of agents is here!", "AI is taking over coding!" and so on and so on. But you've also seen the value of LLMs in coding - they're not a silver bullet, but they've come so far in such a short time that it's impossible to ignore the impact they're having on how we write software. In this series I'll cover some thoughts about how I think software teams can adapt to the new world of *agentic engineering*, challenge some assumptions and hopefully provoke some new thinking about how we can work with these tools to build better software, faster.

There's a lot to cover, so I'm going to split into a mini-series of posts, starting with this one.

## Part 1: AI Made Code Cheap But Delivery Hard

I remember the first time GitHub Copilot autocompleted a line of code for me. It felt magical. Soon I was coding much more than before because I didn't have to remember every little detail or spend time typing up boilerplate code. Soon after, Chat emerged, and coding became more conversational and Copilot could edit multiple files. And soon after that, agents became *de rigueur*, and I am now doing projects (and/or rewrites) that were simply too daunting before.

But I also noticed that more and more developers were struggling to keep up with the pace of the tooling. Companies were also feeling immense pressure to adopt tools that developers insisted on. Management at scale has become a critical problem - and it seems that there is a growing frustration with the lack of **real output** improvement.

Before joining GitHub in 2021, I spent about over a decade as a DevOps consultant. And soon after AI code assistants hit, it seems like we forgot just what made DevOps so critical in the first place. It's becoming clear to me that most of the issues that teams are grappling with today in trying to get a handle on agentic engineering are really just DevOps problems. Some of them look a bit different, but we've focused so much on the AI tooling that we've forgotten about the process and the culture of our teams.

## Pull Request is the new unit of flow

Don't get me wrong - I love these tools (I am completely 100% focused on GitHub Copilot after all)! I can't imagine being a developer without these tools. But if we take a step back and think systemically, *coding was never the biggest bottleneck*. Or it may have been, but (as with everything in DevOps) when you optimize one part of the flow, you shift bottlenecks and constraints to other parts of the system. Now new bottlenecks are popping up - such as *safe delivery*. The cost of code has plummeted, but the cost of bad decisions, weak validation, brittle architecture, and old thinking can easily dilute all the gains agentic engineering can bring.

Agile practices commonly involved breaking work into small, manageable chunks, often called Stories. Each Story would be estimated in terms of Story Points, and teams would plan their work in Sprints, aiming to complete a certain number of Story Points each Sprint. This approach was designed to help teams manage complexity and deliver value incrementally rather than phased like Waterfall, where all the discovery and design were completed up front in hopes of clarifying intent. The aim was to create fast feedback loops that you could use to validate assumptions and adjust course as needed (which is where the term "Agile" comes from). 

But when you're farming work off to swarms of agents in parallel, what meaning do Story Points have? Do we still need Sprints as as synchronization point? How do we redefine productivity now that agents are doing a lot of the coding work? 

Pull Requests (PRs) have always been critical to successful DevOps. This isn't changing - in fact, it's becoming even more important. PRs are, in fact, the **new unit of flow**. When looking at the value stream of software delivery, the PR is the point where all the work of coding, testing, and verification comes together before moving to Production. We need to start measuring and optimizing flow at the PR level - not just from idea to PR, but also from PR to merge - and of course, from merge to production. Coding agents may compress the time it takes to get to a PR, but if the PR review and merge process can't keep up, then you can severely dilute agentic time savings.

## Then vs Now

Why did teams create Sprints anyway? What was the point of (the dreaded) daily standups? A lot of these ceremonies are based on the need for *human coordination.* Humans can't work 24/7, and we have limits on how much context we can hold in our heads at once. Humans are also single-threaded. All of this meant breaking work down into chunks that could be completed by a single developer, and then coordinating "merging" those chunks together at the end of the Sprint (the synchronization point). Story Points and estimates were an attempt to make sure you were able to complete enough for the coordination to work without having to wait at the end of the Sprint for incomplete work to merge.

Agents change this flow. We now have access to asynchronous, continuous PR streams via agents. We can have multiple agents working in parallel on different tasks (or on different variants of the same task!). What should human developers be focusing on? What work is best suited to agents vs humans?

Let's also consider QA. In the old world, QA was often a phase at the end of the Sprint. In the new world, we need to think about how to integrate QA into the flow in a more continuous way. Many teams (but probably not enough) have been "shifting left" with security - we need to do that **and** shift left for quality too. Ending up with a large batch of work waiting for QA or security scanning at the end of the Sprint creates a bottleneck. Instead, we need to think about how to do QA and security in a more continuous way - and this should be built on the CI/CD foundation of DevOps. Automation becomes even more important than before.

We need to "shift left" on governance too. There's a balance between "enterprise alignment" (centralized control) and "team autonomy". Too much enterprise alignment can stifle innovation and prevent teams from accelerating - while too much autonomy can fragment the SDLC, add operational overhead and create risk. 

Before, tribal knowledge was manageable because of the ceremonies like standups and retrospectives. But that doesn't scale for a team of autononomous agents. We need to find new ways to capture and share knowledge across the team. But models (and agents) have context limits, so we can't just "spray and pray" - we have to be targeted with how we handle context.

## New Bottlenecks

These are some of the new bottlenecks that teams are facing in this new world of agentic engineering:

- **Review throughput** - how fast can we review and merge PRs?
- **Test quality** - how reliable are our tests, and how extensive is our test coverage?
- **CI quality** - how much of the verification process is automated, and how reliable is it?
- **Merge latency** - how long does it take for changes to be merged after a PR is opened?
- **Context quality** - how well is context maintained and communicated across the team?
- **Ownership ambiguity** - how clear is it who owns each piece of work?

## Metrics That (Mostly) Matter

Ultimately, if you're not meeting business goals, then it doesn't matter how fast your agents are generating code. Measuring business value delivered is far more important than microscoping agent performance. However, these are some metrics that teams should be paying attention to in this new world:

- **PR size** - larger PRs require more context and more review time, and are more likely to introduce bugs.
- **Agentic PR ratio** - what percentage of PRs are being created by agents vs humans?
- **Review latency** - how long does it take for PRs to be reviewed?
- **Agentic review ratio** - what percentage of PRs reviewed by agents? How successful are those reviews compared to human reviews?
- **Median time-to-merge** - how long does it take for PRs to be merged after being opened?
- **CI pass rate** - how often do changes pass CI before being merged?
- **Agent-authored PR outcomes** - how successful are PRs created by agents in terms of quality and acceptance?

## Dual-Lane Execution

Not all work is suited to agents. Thinking in terms of human and agent lanes can help teams route work to the right executor. Humans excel at navigating ambiguity, making judgment calls, and holding the big picture. Agents can assist with research and planning, but thrive on well-scoped, repeatable tasks where the definition of done is clear and verifiable. Drawing a deliberate line between the two lanes prevents the anti-pattern of dumping everything on agents and hoping for the best, and it keeps humans focused on the high-leverage work that actually moves the product forward.

Of course, these are not set in stone and as agents and models continue to evolve, the boundaries may shift.

| Human Lane | Agent Lane |
|---|---|
| Ambiguity resolution | Bounded fixes |
| Vision | Research, planning |
| Architecture decisions | Test generation |
| Product trade-offs | Refactoring |
| Risk acceptance | Documentation updates |
| Escalation handling | Repetitive toil |
| Assessing business impact | Parallel experiments |

## Conclusion

DevOps has always been about optimizing the flow of software delivery. With the advent of agentic engineering, teams need to double down on good DevOps practices to ensure that the benefits of faster code generation are not lost in bottlenecks around review, testing, and merging. But agentic engineering introduces some fundamentally different opportunities if teams take the time to develop new muscle memory and challenge existing assumptions. 

By focusing on PRs as the unit of flow and optimizing human/agent responsibilities, thinking in terms of async, multi-threaded workflows, teams can adapt to this new world intelligently. In Part 2, we'll dive into how context and policy become critical infrastructure in this new world of agentic engineering. Stay tuned, and...

... happy coding!
