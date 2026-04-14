---
layout: post
title: 'From Sprints to Swarms, Part 3: When Code Gets Cheaper, Judgment Gets More Valuable.'
date: '2026-04-14 09:00:00'
image: /assets/images/2026/04/indy.png
description: >
  In part 3 of this series, I argue that when AI makes implementation cheap, judgment, verification, and architecture discipline become the scarce capabilities that distinguish high-performing teams.
tags:
- ai
- devops
- development
---

1. TOC
{:toc}

In [Part 1]({% post_url 2026/2026-04-03-from-sprints-to-swarms-part-1-ai-made-code-cheap %}) of this series, I argued that AI made code cheap, not delivery easy. In [Part 2]({% post_url 2026/2026-04-07-from-sprints-to-swarms-part-2-context-is-infrastructure %}) I argued that context and policy form the control plane that keeps the speed gains from AI from turning into chaos.

That leaves the last question: when implementation gets dramatically cheaper, what becomes more valuable?

The answer is *judgment*.

Bad decisions, weak proofs, brittle architecture, and slow recovery are still expensive. Agents lower the cost of generating options, but they do not lower the cost of choosing badly or shipping something you cannot safely operate.

This is the shift many teams still underestimate. Cheap generation should not lower the quality bar. It should raise the proof bar. If an agent can produce three plausible implementations before lunch, the hard part is no longer typing one of them. The hard part is deciding which one belongs in your production system, and proving it.

## Cheap Code Changes the Economics

Implementation cost shapes engineering behavior. Teams tolerate awkward abstractions, live with duplicated logic, and defer rewrites because changing the system is expensive. Even when everybody knows a design is wrong, replacement cost often keeps the wrong thing in place.

Agents change that calculation.

You can now run parallel solution probes against the same problem, compare trade-offs quickly, and discard the weaker paths without feeling like you wasted a sprint. Disposable prototypes become practical. Selective rewrites become easier to justify. Branch by abstraction becomes more attractive because trying two implementations behind a stable contract is less painful than it used to be.

That is the upside. The caution is just as important: *the cost of experiments is falling, but the cost of incidents is not*.

Production outages still hurt. Compliance failures still hurt. Data corruption still hurts. Reputational damage still hurts. So while implementation gets cheaper, operational mistakes do not. That means the winning move is not to generate more code. It is to generate more options, then apply better judgment.

This is why I think teams should carry *ideas* forward, not *baggage*. If a prototype taught you something useful, keep the learning. Do not preserve every rushed implementation just because code already exists. Code volume is not an asset by itself. Architecture and intent clarity is.

## Two Lanes, Different Purposes

In the first post I talked about dual-lane execution: humans and agents should not own the same kinds of work in the same way. There is a related distinction inside the codebase itself. Teams increasingly need two lanes for software assets:

| Innovation Lane | Production Lane |
|---|---|
| Fast exploration | Hardened delivery |
| Multiple competing implementations | One supported implementation |
| Loose edges are acceptable | Operational sharp edges are not |
| Local proof may be enough | System proof is required |
| Easy to discard | Expensive to keep |

The innovation lane can be a little messy on purpose. Prototype the workflow. Try two prompts. Compare two parsers. Generate scaffolding. Spike the interface. Throw most of it away if the idea does not hold up.

The production lane is different. Once code graduates into the part of the system that other people depend on, the standards change. At that point, tests are not optional polish. Observability is not nice-to-have. Ownership cannot be fuzzy. Rollback cannot be an afterthought.

That handoff matters more in the agentic era because the front half of the system can now move much faster than the back half. If you do not define graduation criteria, prototype code can easily leak into production simply because it exists and mostly works.

A useful graduation gate asks a few plain questions:

- does this code solve a real, validated problem
- can someone explain the design and its trade-offs clearly
- do tests describe the expected behavior at the right level
- do we know what signals will tell us it is healthy in production
- is there a safe rollback or containment path
- is there a named human owner for the risk

If the answer to those questions is weak, the solution is not ready, *no matter how quickly it was produced*.

## The Developer Role Moves Up the Stack

This is the part that tends to trigger anxiety, but I think the shift is more interesting than threatening. The developer role does not disappear; it shifts toward higher-leverage work.

When code generation is abundant, the highest-value engineering work moves toward direction, selection, and proof. Developers become more like workflow directors than pure implementers. They frame the problem, shape the context packet, set boundaries for the agent, compare candidate solutions, and decide which path deserves deeper investment.

That role has several facets:

- **workflow director** - decomposes work, routes it well, and manages parallel execution
- **context architect** - makes sure the right domain, design, and operational context are available at the start
- **quality governor** - insists on evidence, not just plausible output
- **platform engineer** - improves the paved road so safe agentic delivery is the default
- **reviewer as verifier** - evaluates correctness, risk, and fitness, not just style

None of those are entirely new. But they move much closer to the center of the job.

This is also why I think AI enablement becomes a continuous discipline, not a one-time rollout. Mature teams will keep refining their instructions, task templates, skills, ownership metadata, tests, rulesets, and observability because those things compound. Instructions, examples, docs, and tests are not administrative residue anymore. They are capital that helps humans and agents make better decisions.

In other words, the developer of the next few years is not just writing code. They are designing the system that decides how code gets written, validated, and trusted.

## Verification Becomes a Competitive Advantage

If there is one idea I want to land in this series, it is this: *when generation gets cheap, verification becomes a differentiator*.

That starts with tests, but it does not end there. A strong verification culture treats tests as contracts, observability as proof, rollout telemetry as feedback, and provenance as part of operational reality. It asks not only "did the code compile?" but also "what exactly did we change, why do we believe it is safe, and how quickly will we know if we were wrong?"

That leads to a different kind of handoff. Good handoffs carry evidence:

- what the intended behavior is
- what changed and why
- what tests were run
- what risks remain
- what telemetry or dashboards should be watched
- what the recovery path is if reality disagrees with the plan

This is not bureaucracy. It is how you keep review and operations from collapsing under higher change volume.

It also changes what good platforms optimize for. Faster pipelines matter, but fast wrong answers are not impressive. The real target is trustworthy speed: fast feedback, useful test signals, clean provenance, safe progressive rollout, and recovery paths that work under stress.

That is why cheap code should raise the bar for proof. If your system can generate more change, your verification system has to turn more ambiguity into confidence. Teams that do this well will feel dramatically faster. Teams that do it poorly will feel busy, noisy, and fragile.

## The Platform Matters

This is also why I think an integrated platform matters more in the agentic era.

If agents work in one place, context lives in another, CI runs somewhere else, security signals arrive late, and audit trails have to be stitched together by hand, the system creates friction right where it needs clarity. Review slows down. Provenance gets fuzzy. Policy becomes easier to bypass by accident. The more autonomy you add, the more expensive that fragmentation becomes.

An integrated platform does not remove the need for judgment. It makes judgment easier to apply at the right moment. When code, issues, pull requests, code review, Actions, security controls, environments, and audit history live in the same system, context travels with the work and evidence is easier to inspect.

That is why I think GitHub's Agentic Platform is strategically important here. It gives teams a shared surface for agent execution, human review, workflow automation, and governance rather than forcing them to bolt those capabilities together across disconnected tools. Agents can work where developers already work, and the surrounding controls can stay close to the activity instead of being bolted on later.

That matters because speed is not the hard part anymore. Coordinated, governable, evidence-bearing speed is.

## Better Judgment Beats More Output

Once agents can produce large amounts of software, the hard decisions become more visible.

What is worth building at all? What should be tested as an experiment and then discarded? What should be hardened into a long-lived capability? What should never be automated because the downside of failure is too high? Where does human review add real value, and where is it just ritual?

Those are judgment questions. They live at the boundary between engineering, product, operations, and risk.

This is also where accountability stays stubbornly human. An agent can propose a migration plan, generate a feature, or suggest a test. But a person still owns the decision to ship, the acceptance of residual risk, and the responsibility for the outcome. That is not a limitation of the tools. It is a property of real systems and real organizations.

I also think teams need better trust calibration here. Blind trust is dangerous. Reflexive distrust is wasteful. The useful posture is conditional trust, backed by evidence. Low-risk, well-bounded work with strong automated checks should flow quickly. High-risk work should keep much tighter human loops.

Not every task deserves autonomous parallelization. Agent budget should be treated a bit like compute budget: spend it where speed, coverage, or toil reduction materially improve the business outcome, not where it just creates more artifacts to review.

Autonomy is earned by evidence.

## Metrics That Actually Matter

If you want to know whether judgment and verification are improving, the metrics need to reflect more than raw activity.

A few measures I would watch are:

- **Predicted versus observed productivity** - did the time you thought you saved turn into shipped value, or just more in-flight work?
- **Experiment hit rate** - how often do prototypes or parallel probes produce something worth hardening?
- **Escaped defects** - is faster change creation increasing failure downstream?
- **Rollback frequency and recovery speed** - how often do you need to back out changes, and how quickly can you recover?
- **Review burden** - are senior engineers spending more time triaging noise than evaluating important risk?
- **Time reinvested in architecture and platform** - is AI-created time being spent on higher-leverage work, or just absorbed by more throughput?
- **Trust signals** - do teams increasingly let low-risk work flow through automation because the evidence deserves trust?

Those metrics tell a more useful story than counting prompts, tokens, or lines of generated code. The goal is not to maximize output. The goal is to improve decision quality, delivery quality, and business outcomes.

## DevOps Becomes More Strategic, Not Less

A lot of the market conversation still implies that AI compresses software delivery into a prompt-and-approve loop. I do not buy that. AI mostly exposes whether a team has real delivery discipline underneath the surface.

If your flow is weak, more generation creates more queues. If your context is weak, more autonomy creates more guesswork. If your verification is weak, more output creates more false confidence. If your operational discipline is weak, incidents erase the gains quickly.

That is why I do not think DevOps becomes less important in the agentic era. I think it becomes a strategic differentiator.

The teams that win will not be the teams with the most agents. They will be the teams with the clearest protocols, the best verification culture, the strongest platform leverage, and the most disciplined judgment about where autonomy belongs.

Swarms still need stewards.

## Conclusion

Across this series, I have argued three things. In [Part 1]({% post_url 2026/2026-04-03-from-sprints-to-swarms-part-1-ai-made-code-cheap %}), AI shifted the bottleneck from coding effort to delivery flow. In [Part 2]({% post_url 2026/2026-04-07-from-sprints-to-swarms-part-2-context-is-infrastructure %}), context and policy became the control plane for safe speed. Part 3 is the consequence of both: when code gets cheaper, judgment becomes the scarce capability.

Cheap code is useful. Cheap mistakes are not. The future belongs to teams that can generate options quickly, verify them rigorously, and make disciplined decisions about what deserves to survive. That is less a story about replacing developers than about raising the value of the best parts of engineering judgment.

Happy shipping!
