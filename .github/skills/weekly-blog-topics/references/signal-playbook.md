# Signal-to-Thesis Playbook

Use this after building evidence packets. The question column converts an event into a delivery thesis instead of a release summary.

| Signal in the wild | Question that turns it into a thesis | Likely territory | Likely archetype |
|---|---|---:|---|
| New agent or coding-agent capability ships, from any vendor | Which bottleneck does it move, and which new one does it expose? | 1, 6 | Essay or tutorial |
| DORA, State of DevOps, Accelerate-style, or academic research drops | Does this confirm or contradict a claim Colin already published? | 4 | Commentary |
| Public incident, outage, or breach involves generated code or an agent | Which control-plane layer was absent or ineffective? | 2, 3 | Essay |
| Executive memo, restructure, or layoff is framed around AI | Where do we agree, where do we diverge, and what is non-intuitive? | 5 | Commentary |
| Standards or protocol movement affects MCP, agent identity, SLSA, or attestations | Does this make context or policy more portable, enforceable, or inspectable? | 2 | Essay or tutorial |
| Regulation or compliance requirements change | How must policy-at-runtime change to satisfy the requirement without making review the bottleneck? | 2, 5 | Framework |
| Practitioner essay contradicts an archive thesis | What is the strongest version of that argument, and where is the real disagreement? | Any | Commentary |
| Platform or tooling consolidates | What happens to the paved road, context flow, and evidence trail? | 2, 6 | Essay |
| AI productivity or measurement claim is disputed | What should teams measure instead, and which decision would that metric change? | 4 | Essay or tutorial |
| Agent-generated change volume rises | Which queue grows first: review, CI, merge, deployment, or recovery? | 1, 3 | Essay |
| New context, memory, or instruction mechanism ships | Is context becoming infrastructure, or only a larger prompt surface? | 2, 7 | Commentary or tutorial |
| New agent security control ships | Which threat does it actually contain, and what residual risk stays human-owned? | 2, 3, 6 | Tutorial |

## Pair signals with open threads

The strongest weekly output pairs a verified signal with an explicit open thread. Rank that pairing above equally strong standalone news because it advances the blog's argument rather than restarting it.

Examples:

- A provenance standard can be the occasion to define **evidence-bearing speed**.
- A study of review outcomes can advance the unresolved **future of peer review**.
- New GitHub policy controls can enable the first reproducible **control-plane build**.
- A disputed productivity report can motivate instrumentation for the **six flow metrics**.
- Better task-routing evidence can turn **dual-lane execution** into a decision procedure.

Do not force a weak connection. The evidence must materially help answer the open question.

## Choose the right content unit

Colin sometimes publishes companion pairs: one leadership or framework post and one practitioner tutorial. Propose a pair when the same thesis requires both an operating-model decision and a reproducible implementation.

He also uses short multi-part series. Propose a series when the argument contains distinct constraints that each deserve their own conclusion. Do not use a series merely to inflate one idea.
