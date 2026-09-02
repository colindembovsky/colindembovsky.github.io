# Topic Map

This is the topic radar's centerpiece. Use it after evidence packets exist and before comparing with the archive.

## Core thesis and classifier

Everything should ladder up to **Agentic Software Delivery (ASD)**:

> The practice of integrating *human expertise*, *autonomous AI agents*, and *contextual intelligence* to continuously achieve business outcomes rapidly and securely.

Source: `{% post_url 2025/2025-05-01-agentic-software-delivery %}`.

Classify every candidate against all three pillars:

- **Human Expertise**: judgment, accountability, intent, adoption, culture, and risk acceptance.
- **Autonomous Agents**: delegated execution, orchestration, bounded autonomy, and intelligent automation.
- **Intelligent Context**: instructions, documentation, business intent, policy, provenance, and feedback.

Use HIGH, MEDIUM, or LOW impact exactly as the Eight Principles post does. A candidate must materially affect at least one pillar.

The master frame is: **when code gets cheap, the constraint moves to the delivery system.** A topic is on-lane when it identifies where the constraint moved and what teams should do about it.

## Seven territories

### 1. Delivery flow and bottlenecks

**Covers:** The PR as the unit of flow; idea-to-PR, PR-to-merge, and merge-to-production; review throughput; merge latency; batch versus continuous work; queueing under higher agent PR volume.

**Questions:** Where did the queue move? Which unit best exposes flow? Can review and merge capacity absorb generated changes? Which operational metric reveals the constraint?

**Covered by:** `{% post_url 2026/2026-04-03-from-sprints-to-swarms-part-1-ai-made-code-cheap %}`.

**Saturation:** High on framing, low on instrumentation.

### 2. Context and the control plane

**Covers:** Context packets; documentation as execution input; policy at runtime; custom instructions; Skills; `CODEOWNERS`; rulesets; environment protections; audit trails; provenance; supply-chain risk for agent-authored code.

**Questions:** What must an agent know before acting? Which policies belong at execution time? How do teams make autonomy reviewable, portable, and safe?

**Covered by:** `{% post_url 2026/2026-04-07-from-sprints-to-swarms-part-2-context-is-infrastructure %}`.

**Saturation:** High conceptually, zero hands-on.

### 3. Verification and quality

**Covers:** Test quality versus noise; CI trustworthiness; review by risk rather than ritual; evidence-bearing speed; provenance; verification as competitive advantage.

**Questions:** What proof should travel with a change? How should review depth follow risk? What makes automated evidence trustworthy enough to reduce human review?

**Covered by:** `{% post_url 2026/2026-04-14-from-sprints-to-swarms-part-3-judgment-gets-more-valuable %}`.

**Saturation:** Low. The claims are strong but mostly unoperationalized.

### 4. Measurement and telemetry

**Covers:** Metrics that matter; the anti-vanity-metric stance; agent-authored PR ratio and outcomes; Copilot metrics APIs; DORA in the agent era; Engineering System Success.

**Questions:** Did more output become shipped value? Which measures distinguish acceleration from queue growth? How do human and agent changes compare on flow and outcomes?

**Covered by:** `{% post_url 2026/2026-04-03-from-sprints-to-swarms-part-1-ai-made-code-cheap %}` and `{% post_url 2026/2026-02-02-visualize-copilot-pr-metrics-with-github-app %}`.

**Saturation:** Medium. There is strong appetite but few worked examples.

### 5. Organization design, roles, and economics

**Covers:** Dual-lane execution; workflow director, context architect, quality governor, platform engineer, and reviewer-as-verifier; compensation and incentives; headcount; enablement; adoption curves; culture.

**Questions:** Which roles gain leverage? Which handoffs become expensive? How should incentives reward judgment and outcomes rather than generated output?

**Covered by:** `{% post_url 2026/2026-04-14-from-sprints-to-swarms-part-3-judgment-gets-more-valuable %}`, `{% post_url 2026/2026-05-22-the-100x-org-meets-the-control-plane %}`, and `{% post_url 2025/2025-11-25-building-copilot-enablement-program %}`.

**Saturation:** Medium.

### 6. Automation patterns and Continuous AI

**Covers:** Self-healing pipelines; Agentic Workflows; intent-driven development; Actions plus Models plus MCP; safe outputs; recurring autonomous maintenance.

**Questions:** Which context-dependent toil can become safe continuous automation? What guardrail makes the pattern trustworthy? Is intent a better interface than procedural automation here?

**Covered by:** `{% post_url 2025/2025-08-08-self-healing-devops-with-copilot-and-actions %}` and `{% post_url 2026/2026-02-12-transform-sdlc-with-agentic-workflows %}`.

**Saturation:** Medium. This is the tutorial engine of the blog.

### 7. Human-agent working model

**Covers:** Async-first thinking; "Should Copilot do this?"; delegation heuristics; parallel experiments; fleet orchestration; overcoming "this feels like cheating."

**Questions:** Which work belongs in each lane? How should teams coordinate parallel agents? When must humans stay close to execution?

**Covered by:** `{% post_url 2025/2025-11-25-teaching-async-thinking-with-copilot %}`.

**Saturation:** Low.

## Open-thread backlog

Treat this backlog as the main anti-slop mechanism. A current signal that helps answer one of these questions is usually more valuable than fresh but shallow news.

| Open thread | Territory | Why it remains open | Strong proof that could close it |
|---|---:|---|---|
| Peer review's future | 1, 3, 5 | The 100x Org post calls it "an unresolved architectural question, not a preference." The answer depends on whether agents author at individual or team level. | Comparative team models, risk data, or a concrete review architecture. |
| Instrument the six flow metrics | 1, 4 | Part 1 lists PR size, review latency, median merge time, CI pass rate, agent PR ratio, and agent PR outcomes but implements none. | A reproducible GitHub data pipeline and decision thresholds. |
| Build the control plane | 2, 3 | Part 2 names the stack but does not assemble instructions, context packets, checks, `CODEOWNERS`, rulesets, environment protections, and audit trails. | A public reference repository or step-by-step lab. |
| Deep-dive the five emerging roles | 5, 7 | Part 3 names workflow director, context architect, quality governor, platform engineer, and reviewer-as-verifier without operating models. | One framework per role with responsibilities, measures, and failure modes. |
| Define evidence-bearing speed | 3, 4 | The phrase is asserted but not defined. The archive does not specify evidence, attachment format, evaluator, or confidence threshold. | An evidence contract, PR example, and evaluation procedure. |
| Expand six ASD principles | all | The Eight Principles post gives short GitHub Application stubs; most principles have no dedicated treatment. | A focused framework or tutorial that operationalizes one principle. |
| Agentic Workflow security depth | 2, 3, 6 | Security receives a brief section inside a much broader tutorial. | Threat model, adversarial test, and safe-output controls in a reproducible workflow. |
| Make dual-lane routing decidable | 1, 5, 7 | The archive provides a table but no routing test or process for revisiting the boundary as agents improve. | A decision tree, risk rubric, and reassessment cadence. |

## Refreshing open threads

On each run, scan posts newer than the archive map's `last_updated` value and the newest existing posts. Look for:

- Explicit promises: "in a future post," "I'll cover," "next I will."
- Deliberate gaps: "unresolved," "beyond the scope," "briefly."
- Named concepts with no definition, measure, worked example, or decision procedure.
- Tables or metric lists that state a model without instrumenting it.
- Conclusions that introduce a next constraint but do not answer it.

Add a thread only when the archive genuinely leaves it open. Remove or mark one closed when a newer post answers it.

## Topic rejection tests

Reject a candidate when:

- The post would be a feature summary. "Here is what shipped" is off-lane regardless of the announcement's size.
- It is model capability news without a delivery consequence. Benchmarks, context windows, and price cuts qualify only if they demonstrably move a bottleneck.
- It re-answers an archive question with only the product name changed.
- It concerns a stack Colin does not operate in, such as non-GitHub CI or frontend frameworks.
- It is funding or company drama without a delivery-system consequence.
- It cannot be classified against at least one ASD pillar.
