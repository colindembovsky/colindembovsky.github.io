---
layout: post
title: 'The 100x Org Meets the Control Plane: Reading Zeb Evans Through a DevOps Lens'
date: '2026-05-22 09:00:00'
image: /assets/images/2026/05/control-plane.png
description: >
  Zeb Evans of ClickUp cut 22% of his company and announced million-dollar comp bands for people who build with AI. His memo and my Sprints to Swarms series agree on more than they disagree, and there are some insights on where they complement each other.
tags:
- ai
- devops
- process
---

1. TOC
{:toc}

[Zeb Evans](https://x.com/DJ_CURFEW), CEO of ClickUp, [posted a memo](https://x.com/DJ_CURFEW/status/2057522382315929802) announcing a 22% headcount reduction alongside a new operating model he calls the "100x org," complete with million-dollar salary bands for people who create outsized impact with AI. It is a sharp, opinionated piece. I have been pondering it for a few days, partly because it lands close to the argument I made in my [Sprints to Swarms]({% post_url 2026/2026-04-03-from-sprints-to-swarms-part-1-ai-made-code-cheap %}) series, and partly because where it diverges from that argument is more interesting than where it agrees.

This post is my attempt to compare and contrast the two views and surface the non-intuitive insights that fall out when you put them side by side.

## Where We Agree

The overlap is striking enough that I want to get it out of the way first.

Both Evans and I reject the idea that "more PRs" or "more generated code" is the goal. Evans calls it "the great reckoning of AI coding" and points out that companies celebrating 500% more pull requests are not necessarily shipping 500% more value. I made a similar point in [Part 1]({% post_url 2026/2026-04-03-from-sprints-to-swarms-part-1-ai-made-code-cheap %}): faster code generation does not automatically mean faster delivery, and more code entering the pipe with the same review capacity is a recipe for queues and noise.

We also agree that *judgment* is the new scarce resource. Evans writes that "the skill is judgment" for his 10x engineers. In [Part 3]({% post_url 2026/2026-04-14-from-sprints-to-swarms-part-3-judgment-gets-more-valuable %}) I argued that when code generation (implementation) gets cheap, judgment becomes the differentiator. Different vocabularies, same thesis.

The role shifts line up too. Evans describes engineers who orchestrate, architect, and review agents rather than write code themselves. That maps cleanly to the workflow director, context architect, and reviewer-as-verifier roles I described in Part 3. We both think the developer role moves up the stack and gets more leverage, not less.

And we both believe the existing operating model is the bottleneck. Evans says today's workflows "create bottlenecks in AI systems" and need to be replaced rather than iterated on. I argued that teams adding AI at the edge of the system while leaving the delivery model untouched are predictably stuck. Same diagnosis.

## Where We Diverge

The disagreements are more subtle, but they matter.

Evans frames the problem as an org and compensation redesign. His memo is about who you hire, who you let go, and how you pay the people who stay. My series frames the problem as an engineering and delivery system redesign. I spend most of my time on flow, context, policy, verification, and platform leverage. You have to have both - a redesign of the systems and processes and a redesign of the org and incentives - but they are different angles on the same problem.

That difference shows up most clearly on the topic of review. Evans treats reviewing other people's code as an inefficient bottleneck that wastes elite engineers' time. His preferred model is that a 10x engineer reviews their own agent's output and ships. I treat peer review as essential, but calibrated to risk: low-risk work flows through automation, high-risk work gets deeper human attention. Both positions can be internally consistent, but they imply very different team shapes.

Evans is also far more absolutist about lane boundaries. He flatly states that PMs should not ship production code, because every cross-lane handoff creates review tax on the scarcest resource. I argued for intentional routing between human and agent lanes, but I stopped short of hard organizational boundaries. His version is a stronger form of the same idea. How hard you go on this concept will depend on your company culture and the nature of your product, but it is a real design choice, not a philosophical one.

The largest gap, though, is the control plane. Evans's memo doesn't explicitly mention verification, policy, provenance, recoverey, or governance. [Part 2]({% post_url 2026/2026-04-07-from-sprints-to-swarms-part-2-context-is-infrastructure %}) of my series is almost entirely about those topics. That asymmetry is not a small detail. It changes what the 100x org actually feels like in practice.

## Non-Intuitive Insights

Putting the two views together produced a handful of observations that are unexpected.

### Evans's headcount cut is consistent with his "more people" claim

On the surface, cutting 22% while claiming the 100x org is "infinitely more dependent on people" looks like a contradiction. Read through the lens of dual-lane execution, it stops being one. The roles he removed were largely the ones whose work was already the bottleneck: intermediaries, reviewers of human-authored code, coordination layers that exist to compensate for slow handoffs. The roles he kept and amplified are the ones that do not compress under AI: judgment, customer time, system ownership. He did not shrink the company so much as collapse the lanes where humans were duplicating what agents now do better.

### Peer review's future is an unresolved architectural question, not a preference

Evans and I are both internally consistent, but our positions on peer review depend on an assumption neither of us names. If agents author work at the *individual* level, then Evans is right: peer review becomes the elite engineer reviewing their own agent's output, and external review is friction. If agents author work at the *team* level, where multiple agents and humans contribute to a shared production system, peer review survives as a calibrated risk control. Which world you live in depends on how integrated your codebase is and how much blast radius any single change carries. That is an architectural question linked to risk tolerance, not a management opinion.

### Code volume is a liability at two layers at once

We both say "more code is a bottleneck," but for different reasons. Evans says it wastes elite engineers' attention. I say it overwhelms the delivery system. Stack those together and the claim gets sharper: code volume is a liability at the human-attention layer and the system-throughput layer simultaneously. Optimizing for output is doubly wrong. That is a stronger statement than either of us made alone.

### Dual-lane only works if the lanes are enforced

I described dual-lane execution as *intentional routing*. Evans pushes harder: PMs do not ship to production, full stop. He is applying policy-at-runtime thinking to org design, even though he never uses that language. The non-intuitive read is that my Part 2 argument about putting policy near the activity has a direct organizational analogue. Lanes that are merely suggested decay into informal handoffs and review tax. Lanes that are enforced stay clean.

### A 100x org without a control plane is a 100x incident generator

This is the one I keep coming back to. Evans's comp model rewards output and impact velocity. He does not describe the verification system that decides whether that output is safe to ship. In a world where a small number of elite engineers are directing fleets of agents, a missing control plane is not a minor gap. It is the difference between trustworthy speed and faster risk accumulation. His memo describes the incentives. My series describes the substrate those incentives need in order to produce good outcomes rather than expensive ones.

### The two views are complements, not competitors

Evans is strongest on some topics that I did not cover. He has a sharp economic and organizational thesis: restructure around judgment, pay accordingly, and stop subsidizing the bottlenecks. Where I am strongest, he didn't explictly have opinions. I have an operational thesis: build the context, policy, verification, and platform that make judgment trustworthy at scale. Neither view is complete on its own. Together they form a more honest picture of what the next few years of software delivery look like: *judgment becomes scarce*, so you restructure roles and compensation around it, and you build the control plane that makes that judgment safe to execute on.

## What This Means for Teams Right Now

Here are a few practical takeaways, even for teams that are not about to cut 22% of their workforce.

First, audit which roles in your delivery system exist primarily to compensate for slow handoffs or weak automation. Those are the roles most exposed to the shift Evans is describing, and the most likely to be re-shaped rather than preserved.

Second, be honest about whether your control plane exists. If your verification, policy, ownership, and recovery story is informal, then accelerating generation will accelerate incidents at the same rate. Comp bands and org redesigns will not fix that.

Third, treat lane discipline as a real design choice. Whether or not you go as far as Evans does on PMs and production code, decide deliberately which lanes you enforce and which you only suggest. The suggested ones will probably decay over time.

Finally, do not optimize for output. Optimize for trustworthy, evidence-bearing speed. Evans is right that judgment is the scarce capability. I would add that judgment without a control plane is just confident risk-taking at scale.

## Conclusion

Evans and I are looking at the same shift from different angles. He sees an org and incentive problem and proposes a bold answer. I see an engineering and delivery system problem and propose a change of system and process. Both are right, and both are incomplete without the other. The teams that get this era right will combine the two: restructure around judgment, pay for outsized impact, and build the operational substrate that makes that impact safe, reviewable, and compounding.

Happy shipping!
