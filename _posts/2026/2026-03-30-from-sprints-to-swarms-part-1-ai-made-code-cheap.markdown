---
layout: post
title: 'From Sprints to Swarms, Part 1: AI Made Code Cheap But Delivery Hard.'
date: '2026-03-30 09:00:00'
image: /assets/images/2026/03/sprints-to-swarms-part-1.jpg
image_prompt: "Photo-realistic image of a modern software delivery control room with multiple monitors showing pull requests, CI pipelines, dashboards, and AI coding agents working in parallel, diverse engineering team reviewing flow metrics, natural lighting, focused collaborative atmosphere"
description: >
  Keyword outline for part 1 of the From Sprints to Swarms series, focused on why faster code generation makes DevOps, merge flow, and verification more important than ever.
tags:
- ai
- devops
- process
---

1. TOC
{:toc}

## Series Position

You've heard all the hype: "The age of agents is here!", "AI is taking over coding!" and so on and so on. But you've also seen the value of LLMs in coding - they're not a silver bullet, but they've come so far in such a short time that it's impossible to ignore the impact they're having on how we write software. In this series I'll cover some thoughts about how I think software teams can adapt to the new reality of AI-assisted coding, challenge some assumptions and hopefully provoke some new thinking about how we can work with these tools to build better software, faster.

There's a lot to cover, so I'm going to post three articles:

- Part 1: AI Made Code Cheap But Delivery Hard
  - We'll cover how the economics of software delivery are changing when code generation is no longer the bottleneck, and why that makes DevOps, merge flow, and verification more important than ever.
- Part 2: [Context Is Infrastructure, Policy Is the Runtime]
  - We'll dive into how the shift to agentic software delivery means that context - in the form of documentation, architecture diagrams, ADRs, runbooks, etc - becomes a critical part of the infrastructure that supports safe and efficient delivery, and how policy-as-code and always-on governance are necessary to keep the system running smoothly.
- Part 3: [When Code Gets Cheaper, Judgment Gets More Valuable]
  - Finally, we'll talk about how the role of the developer evolves in this new world, where judgment, verification, and strategic decision-making become more important than ever as the cost of code itself plummets.

## Part 1: AI Made Code Cheap But Delivery Hard

I remember the first time GitHub Copilot autocompleted a line of code for me. It felt magical. Soon I was coding much more than before because I didn't have to remember every little detail or spend time typing up boilerplate code. Soon after, Chat emerged, and coding became more conversational. And soon after that, agents became de rigueur, and I was now trying projects (or rewrites) that were simply too daunting before.

But I also noticed that more and more developers were struggling to keep up with the pace of the tooling. Companies were also feeling immense pressure to adopt tools that developers insisted on. Management at scale became a critical problem - and it seemed that there was a growing frustration with the lack of **real output** improvement.

Before joining GitHub in 2021, I spent about over a decade as a DevOps consultant. And soon after AI code assistants hit, it seems like we forgot just what made DevOps so critical in the first place. It's becoming clear to me that most of the issues that teams are grappling with today in "agentic engineering" are really just DevOps problems. Some of them look a bit different, but we've focused so much on the AI tooling that we've forgotten about the process and the culture of our teams.

## PR is the new unit of flow

Don't get me wrong - I love these tools (I am completely 100% focused on GitHub Copilot after all)! I can't imagine being a developer without these tools. But if we take a step back and think systemically, *coding was never the biggest bottleneck*. Or it may have been, but (as with everything in DevOps) when you optimize one part of the flow, you shift bottlenecks and constraints to other parts of the system. Now *safe delivery* has become the bottleneck. The cost of code has plummeted, but the cost of bad decisions, weak proofs, brittle architecture, and slow recovery can easily dilute all the gains.

Agile practices commonly involved breaking work into small, manageable chunks, often called Stories. Each Story would be estimated in terms of Story Points, and teams would plan their work in Sprints, aiming to complete a certain number of Story Points each Sprint. This approach was designed to help teams manage complexity and deliver value incrementally rather than phased like Waterfall, where all the discovery and design were completed up front in hopes of clarifying intent. The aim was to create fast feedback loops that you could use to validate assumptions and adjust course as needed (which is where the term "Agile" comes from after all). 

But when you're farming work off to swarms of agents, what meaning does a Story point have? Do we still need Sprints as as synchronization point? How do we define productivity now that agents are doing a lot of the coding work? 

The Pull Request (PR) has always been a critical hinge for successful DevOps teams. This isn't changing - in fact, it's becoming even more important. And the PR is the also the **new unit of flow**. When looking at the value stream of software delivery, the PR is the point where all the work of coding, testing, and verification comes together before moving to Production. We need to start measuring and optimizing flow at the PR level - not just from idea to PR, but also from PR to merge - and of course, from merge to production. Coding agents may compress the time it takes to get to a PR, but if the PR review and merge process can't keep up, then you dilute all the time savings.

## Then vs Now

- fixed sprint cadences
- async continuous PR streams
- one developer, one task
- human plus agent parallel execution
- story points
- time-to-merge
- QA after the fact
- always-on governance
- tacit tribal knowledge
- explicit context packets

## New Bottlenecks

- review throughput
- CI capacity
- merge latency
- rework loops
- rollback readiness
- context quality
- flaky tests
- ownership ambiguity

## Metrics That Matter

- PR size
- review latency
- median time-to-merge
- pre-merge CI pass rate
- change failure rate
- MTTR
- reopened PRs
- rollback frequency
- agent-authored PR outcomes

## Dual-Lane Execution

### Human Lane

- ambiguity
- architecture
- product trade-offs
- risk acceptance
- escalation

### Agent Lane

- bounded fixes
- test generation
- refactoring
- documentation updates
- repetitive toil
- parallel experiments

## Practical DevOps Moves

- small batches
- trunk-based development
- merge queue
- branch protection
- reviewer SLAs
- dark launches
- feature flags
- rollback drills
- automated evidence collection

## Anti-Patterns

- velocity worship
- massive AI PRs
- context-free tickets
- end-of-sprint QA
- unowned agent work
- "passed CI means done"

## Closing Beat

- merge flow as heartbeat
- DevOps as control system
- faster generation, tighter controls
- swarms need orchestration
