---
layout: post
title: 'From Sprints to Swarms, Part 2: Context Is Infrastructure, Policy Is the Runtime'
date: '2026-04-06 09:00:00'
image: /assets/images/2026/04/building.png
description: >
  In part 2 of this series, I argue that context and policy form the control plane for agentic delivery. Clear task framing, trustworthy documentation, and policy at the point of execution let teams scale AI speed without scaling chaos.
tags:
- ai
- devops
- security
---

1. TOC
{:toc}

In [episode one]({% post_url 2026/2026-04-03-from-sprints-to-swarms-part-1-ai-made-code-cheap %}), I argued that AI made code cheap, not delivery easy. Once the pull request becomes the unit of flow, the constraint shifts into review, testing, merge speed, and all the coordination work teams used to treat as background noise.

That raises the next question: what does the happy path look like when agents do a meaningful share of the implementation?

It looks something like this. An agent receives a bounded task, finds the right code and docs, makes a small change, runs the right tests, and passes automated checks. A human then reviews the change in proportion to its risk and ships it quickly.

That workflow sounds simple. It only works when two conditions are already in place: reliable context and policy that runs near the activity. Together, they form the control plane for agentic delivery. Without them, faster generation just creates faster review debt.

## The Happy Path Needs More Than a Prompt

A lot of current AI adoption still treats prompting as the main control surface. That is too narrow. The prompt is only the last inch of the pipeline. What matters more is everything upstream that shapes what the agent can see, infer, and safely do.

If you want agents to behave well, you need a codebase and documentation set that are easy to search, easy to trust, and hard to misread. You also need tasks that state intent, scope, constraints, and risk clearly. Otherwise you get the worst kind of acceleration: fast local output followed by slow, expensive review and cleanup.

The happy path still depends on a few things that should be familiar to any team that has done DevOps well:

- clear task intent
- reliable documentation and architecture context
- small batch sizes
- good tests
- clean ownership and review paths
- automated checks, scans, and release protections

None of that is new. What changes is the penalty for getting it wrong. Weak context used to slow people down. In an agentic system, it can create a larger volume of confident mistakes.

## Context Is Infrastructure

Think about what infrastructure as code solved. Before it, admins clicked around consoles, produced inconsistent environments, and created brittle handoffs. Infrastructure as code made infrastructure versioned, testable, and reviewable. That was a major step forward for speed and reliability.

We need the same shift for context. In an agentic system, context is infrastructure. If it is thin, stale, or hard to find, autonomy rests on guesswork.

When a senior engineer joins a task, they do not start from raw code alone. They pull in product intent, domain language, architecture decisions, service boundaries, conventions, known sharp edges, and recent operational history. If your agents are contributing real work, they need the same kind of grounding, delivered in a form they can actually use.

That context usually lives across several layers:

- product intent and acceptance criteria
- domain language and business rules
- ADRs and architecture diagrams
- service boundaries and ownership metadata
- repository instructions and coding conventions
- tests that show expected behavior

Teams can go overboard here. The point is not to dump the entire universe into every task. That just creates noise. The point is to make the right context easy to find and the critical context impossible to miss.

This is where many teams discover that their documentation problem is really a delivery problem. If architectural patterns are not discernible from the code, notes are stale, ownership is ambiguous, or operational constraints only exist in people's heads, the agent does what a human would do under the same conditions: it guesses. The difference is volume and confidence.

## The Context Packet Matters

One practical pattern I like is the context packet. Before work starts, package the task as a small execution contract instead of a vague request.

A good packet answers questions like:

- what outcome are we trying to achieve
- what is explicitly in and out of scope
- what proves the change is done
- what dependencies or affected systems matter
- what risks need special attention
- what roll-forward path exists if this goes badly
- what evidence should accompany the handoff

That sounds close to good issue writing, and it is. The difference is that the packet needs to be written for execution, not just prioritization. A backlog item can tolerate some ambiguity because a human will usually resolve it on the fly. Agents mostly turn that ambiguity into review work.

This is also where Copilot CLI `/research` and `/plan` mode help. `/research` helps gather the relevant code, docs, and constraints before implementation starts. `/plan` forces that context into an explicit approach, which is often the fastest way to expose fuzzy intent before it turns into bad code and noisy review.

This is one of the quieter shifts in the agentic era. Product thinking, architecture thinking, and delivery thinking have to meet earlier. Handoff quality at the start now does a lot to determine how expensive the review and recovery path will be at the end.

## Clean Codebases Are Not Just a Nice-to-Have

This is the unglamorous part, but it matters. If your codebase is hard to understand, your agents will make hard-to-review changes. If it has weak tests, the evidence attached to those changes will be weak too. If ownership is fuzzy, review routing will be fuzzy as well.

A lot of teams get a system to deployable shape, then spend years layering fixes and workarounds instead of improving the structure underneath. That is survivable in a human-only model. It gets much more expensive when you add agents that can produce changes faster than the system can absorb them.

For some teams, the first serious agentic investment should not be net-new feature work. It should be test expansion, modular refactoring, better ownership metadata, and clearer repository guidance. Those are not side quests. They are how you build the context infrastructure that makes autonomy safer.

Once that foundation improves, the payoff compounds. Agents can navigate the system more accurately, reviewers can reason about changes faster, and the codebase becomes a source of truth instead of a source of confusion.

## Documentation Is Now an Execution Input

I think this is the point many teams still underestimate. Documentation gets treated as onboarding material or compliance residue. In an agentic system, it is also an execution input.

That includes obvious things like API contracts and architecture notes, but it also includes all the little details that experienced teams accumulate over time: naming conventions, test expectations, release rules, migration patterns, and operational caveats. If those things are not discoverable, agents will either miss them or infer them poorly.

DORA has written about [documentation quality](https://dora.dev/capabilities/documentation-quality/) as a capability amplifier, and that framing fits here. Good documentation does not just help people ramp up. It improves change quality because the system becomes easier to understand and safer to modify.

The reverse is also true. Doc rot is not a cosmetic issue. It is a delivery failure mode. If the docs say one thing and the code or platform behavior says another, review slows down, false confidence rises, and both humans and agents learn not to trust the written system.

That is why I think mature teams will maintain context the way they maintain tests: continuously, as part of the normal flow of change.

> Note: Agents can help here too. The [Documentation Updater sample](https://github.github.com/gh-aw/setup/creating-workflows/#github-web-interface) is a practical pattern for keeping docs aligned with code changes.

## Policy Is the Runtime

Context alone is not enough. Even perfect documentation does not replace guardrails.

In a human-only workflow, policy often lives too far from the moment of execution. You see it in tribal review norms, security checklists, end-of-release approvals, or an outdated wiki page nobody opens until something goes wrong. That model does not scale when change volume rises.

Policy has to run where the work happens.

In practice, that means using [repository rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets), [required checks](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches#require-status-checks-before-merging), [CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners), [secret scanning](https://docs.github.com/en/code-security/concepts/secret-security/about-secret-scanning), [code scanning](https://docs.github.com/en/code-security/concepts/code-scanning/about-code-scanning), [dependency review](https://docs.github.com/en/code-security/concepts/supply-chain-security/about-dependency-review), [environment protections](https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments#deployment-protection-rules), [audit logs](https://docs.github.com/en/enterprise-cloud@latest/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/audit-log-events-for-your-organization), and CI/CD automation that enforces the standards the team claims to care about.

This is not about distrusting developers or agents. It is about reducing variance. Good policy removes low-value decisions from the critical path and reserves human judgment for the places where judgment actually matters.

That distinction matters. If every PR gets the same human scrutiny regardless of risk, your reviewers become the bottleneck and the review signal gets diluted. If low-risk work can flow through strong automated controls, humans can spend their time where the blast radius is real.

## Review by Risk, Not Ritual

The old habit is ritual: every change gets the same ceremony. The better habit is calibration: review in proportion to risk.

A low-risk documentation fix should not wait in the same queue or require the same depth of analysis as an auth change, a payment flow change, or infrastructure that can disrupt production. Agentic delivery makes that distinction more important because the system can generate a lot of small, safe changes alongside a smaller number of high-consequence ones.

Teams need an explicit risk model:

- every PR gets Copilot Code Review plus baseline checks
- low-risk work gets fast automated validation and lightweight review, or automatic approval where policy allows
- medium-risk work gets normal peer review plus targeted integration tests
- high-risk work gets deeper human review, stronger evidence, and tighter deployment controls

The point is not bureaucracy. It is calibration. You want the amount of human attention to match the potential downside of the change.

This is also where evidence-bearing handoffs matter. A good handoff does not just say, "I changed some files, please review." It says: here is the intent I believe I implemented, here are the files I changed, here are the tests I ran, here are the risks I still see, here is how to roll it back, and here is what I did not do from the acceptance criteria. That turns review from archaeology into decision-making.

## The Control Plane in GitHub Terms

Teams should think of this as a layered control plane.

Copilot custom instructions, custom agents, and skills shape how work gets interpreted and executed. Issues and pull request templates provide the context packet. Actions and required checks verify behavior continuously. CODEOWNERS and ownership metadata route the work to the right humans. Repository rulesets enforce the non-negotiables. Environment protections and deployment approvals contain higher-risk changes. Audit logs and PR history preserve the evidence trail.

None of these features matter in isolation. The value comes from how they combine. The agent gets better context at the start, the system enforces better policy during execution, and the reviewer receives a smaller, clearer unit of work with better evidence attached.

The more autonomous execution becomes, the more deliberate the surrounding system has to be. Speed without a control plane is just faster risk accumulation.

## New Operating Rituals

Once you accept that context and policy are part of the runtime, a few rituals start to change.

Standups matter less as status broadcasts and more as exception handling. Review queues need active triage because review latency becomes a first-order constraint. Retrospectives should look at flow, false positives, noisy checks, stale docs, and policy gaps, not just missed estimates. Teams also need a visible backlog for context maintenance, because docs and instructions decay unless someone owns that work.

## Conclusion

In Part 1, I argued that AI changes where the bottlenecks live. Part 2 is the operational consequence of that shift. If agents are going to produce meaningful amounts of code, context stops being background material and policy stops being a late-stage gate. They become infrastructure and runtime.

Better context improves autonomy. Stronger policy makes speed safer. Teams that build this control plane will scale agentic delivery far more effectively than teams that simply add more agents to a messy system.

In Part 3, I'll look at the next implication: when code gets cheaper, judgment becomes the scarce capability.

Happy shipping!
