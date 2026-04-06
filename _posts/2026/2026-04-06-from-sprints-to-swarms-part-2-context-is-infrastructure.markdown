---
layout: post
title: 'From Sprints to Swarms, Part 2: Context Is Infrastructure, Policy Is the Runtime.'
date: '2026-04-06 09:00:00'
image: /assets/images/2026/04/sprints-to-swarms-part-2.jpg
image_prompt: "Photo-realistic image of software engineers working with architecture diagrams, documentation, policy dashboards, and GitHub repository controls on large screens, AI agents operating within visible guardrails, modern office, crisp natural light, calm high-trust atmosphere"
description: >
  In part 2 of this series, I argue that context and policy form the control plane for agentic software delivery. Good task framing, reliable documentation, and policy at the point of action let teams scale AI speed without scaling chaos.
tags:
- ai
- devops
- security
---

1. TOC
{:toc}

In [episode one]({% post_url 2026/2026-04-03-from-sprints-to-swarms-part-1-ai-made-code-cheap %}), I argued that AI made code cheap but did not make delivery easy. Once the pull request becomes the unit of flow, the pressure moves into review, testing, merge speed, and all the coordination work we used to treat as background noise.

That creates a new question: what does the happy path look like when agents are doing a meaningful share of the implementation? If we have that picture, teams can start to build the systems and habits that make it more likely.

The happy path is simple to describe. An agent receives a clear task, grounds itself in the relevant parts of the codebase and docs, makes a bounded change, runs the right tests, and undergoes automated quality gates, checks and agentic reviews. Then a human reviews the change and risk quickly, shipping a safe and reliable update.

That sounds straightforward, but it only works if two things are already true. First, the system has reliable context. Second, the system has policy that runs near the action instead of weeks later in a meeting, checklist, or audit. That is the control plane for agentic delivery, and it is why context and governance have to become first-class engineering concerns.

## The Happy Path Needs More Than a Prompt

A lot of current AI adoption still treats prompting as the main control surface. That is too narrow. A prompt is just the last inch of the pipeline. What matters more is everything upstream that shapes what the agent can see, infer, and safely do.

If you want agents to behave well, you need a codebase and documentation set that are easy to search, easy to trust, and hard to misread. You also need tasks that tell the truth about intent, scope, and risk. Otherwise you get the worst kind of acceleration: fast local output followed by slow, expensive review and cleanup.

The happy path depends on a few items that should be familiar to any team that has done DevOps well:

- clear task intent
- reliable documentation and architecture context
- small batch sizes
- good tests
- clean ownership and review paths
- automated checks, scans, and release protections

None of that is new to DevOps. What is new is the consequence of getting it wrong. With human-only delivery, weak context slowed people down. With agentic delivery, weak context can create a larger volume of confident mistakes.

## Context Is Infrastructure

Do you remember when cloud admins clicked around their cloud interfaces to deploy infrastructure for applications - and what a nightmare that was? Manually configuring infrastructure is not scalable, consistent and is highly error prone. This gave rise to infrastructure as code, which made infrastructure more like software: versioned, testable, and reviewable. That was a huge step forward for reliability and speed.

In the agentic era, we need to do the same thing for context. Context is the infrastructure for safe, reliable agentic delivery. If you do not have good context, you do not have a good foundation for autonomy.

When a senior engineer joins a task, they do not start from raw code alone. They pull in product intent, domain language, architecture decisions, service boundaries, conventions, known sharp edges, and recent operational history. If your agents are contributing real work, they need the same kind of grounding, delivered in a form they can actually use.

That context usually lives across several layers:

- product intent and acceptance criteria
- domain language and business rules
- ADRs and architecture diagrams
- service boundaries and ownership metadata
- coding conventions and repository instructions
- clear coding conventions and architecture within the codebase

Teams can go overboard here: the point is not to dump the entire universe into every task. That just creates noise. The point is to make the right context easy to find and the critical context impossible to miss.

This is where many teams discover that their documentation problem is really a delivery problem. If architectural patterns are not discernable from the code, notes are stale, ownership is ambiguous, or operational constraints only exist in people's heads, the agent does what a human would do under the same conditions: it guesses. The difference is that it can guess much faster (and much more confidently) and at much larger scale.

## The Context Packet Matters

One practical pattern I like is the context packet. Before work starts, package the task as a small execution contract rather than a vague request.

A good packet answers questions like:

- what are we trying to achieve
- what is explicitly in and out of scope
- how will we know the change is done
- what dependencies or affected systems matter
- what risks need special attention
- what roll-forward path exists if this goes badly
- what evidence should accompany the handoff

That sounds close to good issue writing, and it is. The difference is that the packet needs to be written for execution, not just prioritization. A backlog item can get away with a little ambiguity because a human will usually resolve it on the fly. An agent needs the ambiguity reduced earlier or it will create work for review instead of reducing it.

This is one of the quiet shifts in the agentic era. Product thinking, architecture thinking, and delivery thinking have to meet earlier. The handoff quality at the start now determines how expensive the review and recovery path will be at the end.

## Clean Codebases Are Not Just a Nice-to-Have

This is the most basic part of context, but it is worth saying. If your codebase is a mess, your agents will be too. If your codebase has no tests, your agents will have no tests. If your codebase has no clear ownership, your agents will have no clear ownership.

Most codebases take about 6 months or so to go from nothing to a deployable state. Thereafter, codebase quality plateaus. Most teams spend a lot of time putting chewing gum onto duct tape to keep the system running, but they do not spend as much time improving the underlying code quality. That is a problem when you want to add agents into the mix. If the codebase is already hard to understand and change, adding more change agents just makes it worse.

Teams have also been reluctant to throw the code away entirely, take forward the learnings, and start clean. But that may be the exact task you need to start your agentic engineering journey on. Get the agents to create a comprehensive test suite, refactor the codebase into clearer modules, and add ownership metadata. That is a great way to build the context infrastructure while also improving the code quality. They can do this in the background for you.

Once you've cleaned your codebase, you have a much stronger foundation for autonomy. The agents can understand the system better, make safer changes, and generate more value with less human intervention. The codebase becomes a source of truth rather than a source of confusion.

## Documentation Is Now an Execution Input

I think this is the point many teams still underestimate. We talk about documentation as if it were a nice-to-have artifact for onboarding or compliance. In an agentic system, documentation is increasingly an execution input. This is not simply a check-mark activity any more: it is the difference between a system that can learn and adapt and a system that just guesses and hopes for the best.

That includes obvious things like API contracts and architecture notes, but it also includes all the little details that experienced teams accumulate over time: naming conventions, test expectations, release rules, migration patterns, and operational caveats. If those things are not discoverable, agents will either miss them or infer them poorly.

DORA has written about [documentation quality](https://dora.dev/capabilities/documentation-quality/) as a capability amplifier, and that framing fits here. Good documentation does not just help people learn faster. It makes every other capability work better because the system becomes easier to understand and change safely.

The reverse is also true. Doc rot is not a cosmetic issue. It is a failure mode. If the docs say one thing and the code or platform behavior says another, you will burn time in review, increase false confidence, and train both humans and agents not to trust the written system. Once that trust drops, throughput drops with it.

That is why I think mature teams will maintain context the way they maintain tests: continuously, as part of the normal flow of change.

> Teams should also use agents to keep documentation up to date! I highly recommend using the [Documentation Updater sample](https://github.github.com/gh-aw/setup/creating-workflows/#github-web-interface).

## Policy Is the Runtime

Context alone is not enough. Even perfect documentation does not replace guardrails.

In a human-only workflow, policy often lives too far from the moment of execution. You see it in tribal review norms, security checklists, end-of-release approvals, or an outdated wiki page nobody opens until something goes wrong. That model does not scale when change volume rises.

Policy has to run where the work happens.

In practice, that means using [repository rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets), [required checks](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches#require-status-checks-before-merging), [CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners), [secret scanning](https://docs.github.com/en/code-security/concepts/secret-security/about-secret-scanning), [code scanning](https://docs.github.com/en/code-security/concepts/code-scanning/about-code-scanning), [dependency review](https://docs.github.com/en/code-security/concepts/supply-chain-security/about-dependency-review), [environment protections](https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments#deployment-protection-rules), [audit logs](https://docs.github.com/en/enterprise-cloud@latest/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/audit-log-events-for-your-organization), and end-to-end CI/CD automation so the system enforces what the team already says it cares about.

This is not about distrusting developers or agents. It is about reducing variance. Good policy removes low-value decisions from the critical path and reserves human judgment for the places where judgment actually matters.

That distinction matters. If every PR gets the same human scrutiny regardless of risk, your reviewers become the bottleneck and the review signal gets diluted. If low-risk work can flow through strong automated controls, humans can spend their time where the blast radius is real.

## Review by Risk, Not Ritual

The old habit is to review because process says every change must go through the same ceremony. The better habit is to review according to risk.

A low-risk documentation fix should not wait in the same queue or require the same depth of analysis as an auth change, a payment flow change, or infrastructure that can disrupt production. Agentic delivery makes that distinction more important because the system can generate a lot of small, safe changes alongside a smaller number of high-consequence ones.

Teams need an explicit risk model:

- every PR gets a Copilot Code Review check (enabled at scale at the organization level as a policy)
- low-risk work gets fast automated validation and lightweight review (or even automatic approval in some cases)
- medium-risk work gets normal peer review plus targeted integration tests
- high-risk work gets deeper human review, stronger evidence, and tighter deployment controls

The point is not bureaucracy. It is calibration. You want the amount of human attention to match the potential downside of the change.

This is also where evidence-bearing handoffs matter. A good handoff does not just say, "I changed some files, please review." It says: here is the intent I believe I implemented, here are the files I changed, here are the tests I ran, here are the risks I still see, here is how to roll it back, and here is what I did not do from the acceptance criteria. That turns review from archaeology into decision-making.

## The Control Plane in GitHub Terms

Teams should think of this as a layered control plane.

Copilot custom instructions, custom agents and skills shape how work gets interpreted and executed. Issues and pull request templates provide the context packet. Actions and required checks verify behavior continuously. CODEOWNERS and ownership metadata route the work to the right humans.Repository rulesets and hooks enforce the non-negotiables. Environment protections and deployment approvals contain higher-risk changes. Audit logs and PR history preserve the evidence trail.

None of these features matter in isolation. The value comes from how they combine. The agent gets better context at the start, the system enforces better policy during execution, and the reviewer receives a smaller, clearer unit of work with better evidence attached.

The more autonomous the execution becomes, the more deliberate the surrounding system has to be. Speed without a control plane is just faster accumulation of risk.

## New Operating Rituals

Once you accept that context and policy are part of the runtime, a few rituals start to change too.

Standups matter less as status broadcast and more as exception handling. Review queues need active triage because review latency becomes a first-order constraint. Retrospectives should look at flow, false positives, noisy checks, stale docs, and policy gaps, not just missed estimates. And teams need a visible backlog for context maintenance, because if docs and instructions never get refreshed, the whole system decays.

## Conclusion

In Part 1, I argued that AI changes where the bottlenecks live. Part 2 is the operational consequence of that shift. If agents are going to produce meaningful amounts of code, then context stops being background material and policy stops being a late-stage gate. They become infrastructure and runtime.

Better context produces better autonomy. Stronger policy produces safer speed. Teams that build this control plane will scale agentic delivery far more effectively than teams that simply add more agents to a messy system.

In Part 3, I’ll look at the next implication: when code gets cheaper, judgment becomes the scarce capability.

Happy shipping!
