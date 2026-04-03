---
layout: post
title: 'From Sprints to Swarms, Part 1: AI Made Code Cheap But Delivery Hard.'
date: '2026-04-03 09:00:00'
image: /assets/images/2026/04/carriage.png
description: >
  In part 1 of this series, I argue that AI made code generation cheap while shifting the real constraint to delivery. Pull requests become the unit of flow, exposing pressure on review, testing, merge speed, and how humans and agents share work.
tags:
- ai
- devops
- process
---

1. TOC
{:toc}

We all know that software engineering has been irrevocably changed by LLMs and agents. It is a mistake to think the main change is *faster typing*. It isn't. The bigger change is economic: when code gets cheaper, the delivery system becomes the constraint.

This series is about what that means for software teams. Part 1 is about flow. Part 2 will cover context and policy. Part 3 will cover judgment and the changing role of the developer.

## Part 1: AI Made Code Cheap But Delivery Hard

Copilot, Chat, and now agents have expanded the amount of software I can produce. They let me tackle work I would have avoided a few years ago because it was too laborious, too time consuming. But they have also exposed something uncomfortable: faster code generation does not automatically mean faster delivery.

Before joining GitHub in 2021, I spent more than a decade in DevOps consulting. From that angle, most "agentic engineering" problems look familiar: they are still about process and culture more than about the tools. The labels may have changed, but most of the constraints have not. If code is cheaper, then review quality, test quality, merge flow, context management and architecture discipline matter even more.

## The Pull Request Is the Unit of Flow

Agile teams broke work into stories, estimated them with story points, and synchronized through sprints. That made sense when the system was shaped around human constraints: people are single-threaded, context is limited, and coordination is expensive.

Agents weaken those assumptions. Work can now be decomposed and executed in parallel, often asynchronously. That shifts the real unit of flow from the story to the pull request.

Teams should measure flow at the PR level: idea-to-PR (how fast can we code it), PR-to-merge (how fast can we validate it), and merge-to-production (how fast can we deliver it).

## Then vs Now

Sprints, standups, and estimates are coordination tools. They help humans batch work, surface blockers, and synchronize periodically.

Agentic delivery looks different. You can have a continuous stream of PRs from multiple agents working in parallel. That changes the role of the human. The highest-value work shifts toward framing problems, resolving ambiguity, setting policy, reviewing risk, and deciding what is worth shipping.

It also breaks batch-oriented quality practices. QA cannot stay at the end of the sprint. Security cannot be a late gate. Governance cannot depend on tribal knowledge. Quality, policy, and context have to move earlier and run continuously.

This is where many teams get stuck. They add AI at the edge of the system, but leave the delivery model untouched. The result is predictable: more code entering the pipe, with the same review capacity and the same weak automation.

## New Bottlenecks

The bottlenecks do not disappear. They move:

- **Review throughput** - can review capacity keep pace with PR volume?
- **Test quality** - do tests catch regressions without creating noise?
- **CI quality** - is verification fast, trustworthy, and automated?
- **Merge latency** - how long do approved changes sit before merge?
- **Context quality** - do people and agents have the right information at the right time?
- **Ownership clarity** - who decides, reviews, and accepts risk?

## Metrics That Matter

Business outcomes matter more than agent benchmarks. If faster code does not produce better delivery, the optimization is irrelevant. Start with a few operational metrics that expose flow quality:

- **PR size** - larger PRs increase review cost and defect risk
- **Review latency** - how long it takes a PR to get meaningful attention
- **Median time-to-merge** - the clearest signal of delivery friction
- **CI pass rate** - whether your verification system is doing its job
- **Agent-authored PR ratio** - how much work is entering the system through agents
- **Agent-authored PR outcomes** - whether that work is actually getting accepted and shipped

## Dual-Lane Execution

Not all work should go to agents. A useful model is dual-lane execution: one lane for human judgment, one for agent execution.

Humans are better at ambiguity, trade-offs, architecture, and accepting risk. Agents are better at bounded fixes, repetitive refactors, test generation, documentation updates, and parallel experiments.

The point is not rigid separation, but intentional routing. Agents simply cannot do everything, despite the hype. But we can probably give more work than we realize to agents. Good teams decide which lane owns which kind of work, then revisit that split as the agents evolve. Here is a rough sketch of how that might look:

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

AI did not remove the need for DevOps. If anything, it increased it. When code gets cheaper, flow discipline, verification, and judgment become more important, not less.

Treat the PR as the unit of flow. Design for continuous review and verification. Be explicit about which work belongs to humans and which belongs to agents. In Part 2, I'll look at the next constraint: context and policy as operational infrastructure.

Happy shipping!
