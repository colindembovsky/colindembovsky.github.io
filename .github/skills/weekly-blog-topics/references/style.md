# Editorial Style and Archetypes

Load this only after a topic has passed evidence, eligibility, archive comparison, scoring, and diversity reranking. It shapes a strong thesis; it must not manufacture one.

## Identity and positioning

- **Site:** Colin's ALM Corner.
- **Tagline:** "DevOpsology & GitHub. *Opinions my own*."
- **Lane:** DevOps tooling, culture, philosophy, GitHub, and Agentic Software Delivery.
- **Position:** GitHub employee since 2021 and DevOps consultant for more than a decade before that.
- **Default lens:** Agentic-engineering problems are usually process and culture problems wearing new labels.
- **Audience:** Competent engineering leaders and practitioners who need decisions, operating models, measures, or reproducible patterns.

## Voice

- Use first person for Colin's argument or experience: "I argue," "I have been pondering," and "In this post, I'll show you."
- Prefer plain, declarative sentences and define a buzzword immediately when it earns a place.
- Lead with the economic reframe: when code gets cheaper, the delivery system becomes the constraint.
- Use "not X, but Y" and "the bottlenecks do not disappear, they move" when they clarify a real contrast, not as decoration.
- Be measured and contrarian. Push back on hype without cynicism.
- Favor process and culture over tool novelty.
- Present GitHub as capability, not marketing. State limitations and counterarguments.
- End optimistically and never condescend.

## Recurring vocabulary

Use only when the new thesis advances it:

- Agentic Software Delivery and its three pillars
- control plane
- dual-lane execution
- PR as the unit of flow
- context is infrastructure; policy is the runtime
- context packet
- review by risk, not ritual
- judgment as the scarce capability
- Continuous AI
- intent-driven development
- async-first and multi-threaded development
- workflow director, context architect, quality governor, platform engineer, reviewer-as-verifier
- verification as competitive advantage
- evidence-bearing speed

## Five archetypes

### Hands-on build or tutorial

**Use when:** The thesis can be proved through a public, reproducible build.

**Skeleton:** Why this matters -> Prerequisites -> How it works -> implementation -> annotated Notes -> Tips and Gotchas -> Conclusion.

**Archive examples:** `{% post_url 2025/2025-08-08-self-healing-devops-with-copilot-and-actions %}`, `{% post_url 2026/2026-02-02-visualize-copilot-pr-metrics-with-github-app %}`, `{% post_url 2026/2026-02-12-transform-sdlc-with-agentic-workflows %}`.

### Opinion or thesis essay

**Use when:** Evidence reveals a changed constraint that needs an economic or operating-model reframe.

**Skeleton:** Economic reframe -> named concept -> then-versus-now contrast -> bottlenecks -> useful metrics -> comparison table -> Conclusion.

**Archive examples:** The three `{% post_url 2026/2026-04-03-from-sprints-to-swarms-part-1-ai-made-code-cheap %}` series posts.

### Framework or numbered principles

**Use when:** Readers need a decision model, role definition, adoption program, or repeatable operating practice.

**Skeleton:** "X isn't enough" premise -> numbered principles or patterns -> Putting It All Together -> Common Pitfalls -> Getting Started -> Conclusion.

**Archive examples:** `{% post_url 2025/2025-08-11-eight-principles-agentic-software-delivery %}`, `{% post_url 2025/2025-11-25-building-copilot-enablement-program %}`, `{% post_url 2025/2025-11-25-teaching-async-thinking-with-copilot %}`.

### Industry commentary or response

**Use when:** A memo, report, incident, or announcement creates a genuine tension with the archive.

**Skeleton:** Where We Agree -> Where We Diverge -> Non-Intuitive Insights -> What This Means for Teams Right Now -> Conclusion.

**Archive example:** `{% post_url 2026/2026-05-22-the-100x-org-meets-the-control-plane %}`.

### Event or talk recap

**Use when:** Colin has a real public talk or recording to share. Do not propose this archetype based on news alone.

**Skeleton:** Brief context -> embedded recording -> session overview -> What's Next? -> Conclusion.

**Archive example:** `{% post_url 2025/2025-12-09-agentic-future-aws-reinvent %}`.

## Drafting conventions appendix

- Path: `_posts/YYYY/YYYY-MM-DD-kebab-case-title.markdown`.
- Front matter: `layout: post`, title, `date: 'YYYY-MM-DD 09:00:00'`, image path, optional `image_prompt`, 1-2 sentence description, and 1-4 established tags.
- Open the body with `1. TOC` followed by `{:toc}`.
- Add `## Series` with sibling `{% post_url %}` links for series posts.
- Close with `## Conclusion` and a topic-matched `Happy <gerund>!`.
- Do not use em dashes. Use spaced hyphens.
- Cross-link relevant prior posts with exact `{% post_url %}` identifiers.
- Wrap code containing GitHub Actions `${{ }}` syntax in Liquid `{% raw %}` and `{% endraw %}` tags.
- Tutorials should explain larger code blocks with numbered or line-range `Notes:`.
- Use tables for useful two-column contrasts.
