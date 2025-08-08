## Purpose

These instructions teach an assistant to draft new blog posts that match this site’s established voice and formatting, derived solely from existing content in `_posts`.

## Voice and point of view

- Tone: friendly, confident, pragmatic, and actionable. Avoid fluff and hype.
- POV: primarily second person (“you”), with occasional first person (“I”) for experience, opinions, and anecdotes.
- Style: explain the “why,” then show the “how.” Prefer concrete steps, code, and configuration over abstract advice.
- Keep it concise, but complete. Use contractions. A touch of personality is fine.
- Close with a short, upbeat sign‑off that fits the topic (e.g., “Happy building!”).
- DO NOT USE m-dash (—) characters.

## Post structure (high level)

1) Brief intro: what the reader will achieve and why it matters.
2) Table of contents (use the site’s TOC convention below).
3) Context and prerequisites (tools, versions, assumptions).
4) Step‑by‑step implementation with code/config, screenshots/diagrams as needed.
5) Notes/tips/caveats and alternatives.
6) Conclusion with 1–2 sentence recap and a sign‑off.

## Front matter and boilerplate

- Use layout `post`.
- Title in Title Case, concise and specific.
- Date string in the site’s style: `'YYYY-MM-DD HH:MM:SS'`.
- Tags: 1–4 lower‑case keywords already used on the site (examples seen: `ai`, `actions`, `alm`, `analytics`, `appinsights`, `automation`, `azure`, `build`, `cloud`, `development`, `devops`, `docker`, `github`, `process`, `security`, `sourcecontrol`, `testing`).
- Include a prompt for the hero image that can be used to generate a photo-realistic AI image

Example:

```markdown
---
layout: post
title: My Specific, Benefit‑Driven Title
date: '2025-08-08 01:22:01'
image: /assets/images/2025/08/example/hero.jpg
image_prompt: "<the prompt that can be used to generate the AI image>"
description: >
  One‑sentence practical summary of what the reader will learn and why it’s useful.
tags:
- github
- security
---

1. TOC
{:toc}
```

## Formatting conventions

- Insert the TOC exactly as:
  - `1. TOC` then on the next line `{:toc}`
- Use `##` and `###` headings. Keep headings short and descriptive.
- Use fenced code blocks with language hints: `yml`, `yaml`, `bash`, `powershell`, `json`, `csharp`, etc.
- When embedding YAML that could trigger Liquid parsing, wrap code fences with `{% raw %}` and `{% endraw %}` inside the fence as shown in many posts.
- Use inline code for CLI flags, environment variables, file names, and identifiers.
- Callouts/tips: start a paragraph with `> Note:` (or short quotes as appropriate).
- Images: attach with center styling and a caption line immediately after.
  - Example:
    - `![Alt text](/assets/images/2025/08/example/screenshot.png){: .center-image }`
    - `Descriptive caption.`
    - `{:.figcaption}`
- Lists: prefer ordered lists for sequences; unordered for bullets. Short sentences per bullet.
- Link to official docs or your repos inline. Use descriptive link text.

## Content patterns to emulate

- Set context fast: summarize the problem and why the reader should care (business/quality/velocity/security angle where relevant).
- Show the “contract” up front for technical steps (inputs, outputs, success criteria, common errors).
- Provide stepwise instructions with minimal, copy‑friendly commands and code.
- After each larger code snippet, briefly explain what it does and any knobs to change.
- Include small “Notes:” sections for gotchas (versions, permissions, environment quirks).
- Use examples grounded in GitHub, Azure, DevOps, security, testing, or build topics consistent with existing posts.
- Finish with a short conclusion and a “Happy …” sign‑off tailored to the topic (e.g., “Happy scanning!”, “Happy federating!”, “Happy source controlling!”).

## Do and don’t

Do:
- Keep titles clear and specific; avoid clickbait.
- Prefer real commands, configs, and screenshots over theory.
- Use the site’s caption/TOC/image conventions.
- Cite sources or link to docs when asserting behaviors or limits.
- Use American English spelling and contractions.

Don’t:
- Don’t pad with generic intros or definitions the audience already knows.
- Don’t invent features, APIs, or screenshots.
- Don’t overuse emojis or exclamation marks (except the closing “Happy …!” is fine).
- Don’t stray into unrelated topics; keep scope tight.

## Reusable section snippets

Intro (pattern):
- “In this post, I’ll show you how to <do X> and why it matters for <value: speed, quality, security, reliability>.”

Prerequisites (pattern):
- “You’ll need <tools/versions/permissions>. Assumptions: <list>. If you don’t have <X>, see <link>.”

Notes (pattern):
> Note: <1–3 line caveat or tip>

Conclusion (pattern):
- “<One‑sentence recap>. <Optional next step/variation>. Happy <verb‑ing>!”

## Images and attribution

- Include a prompt that can be used to generate a hero image later.
- Prefer diagrams/screenshots that add instructional value.

## Tag guidance

- Choose 1–2 tags already present on the site. Favor topic + platform (e.g., `security`, `actions`, `github`, `azure`).

## Code Block explanations
- After a code block, explain the blocks of code using line annotations like this:
```yml
Notes:
  - Line 2: We pass in the $targetPath – this is the path on the target server we want to perform tf commands from
  - Line 3: We in $tfArgs – these are the arguments to pass to tf.exe
  - Line 7-8: get the path to tf.exe and store it
  - Line 10-12: if the $targetPath does not exist, create it
  - Line 14: change directory to the $targetPath
  - Line 15: Invoke tf.exe passing the $tfArgs we passed in as parameters
  - Line 17-19: Since this script invokes tf.exe, you could get a failure from the invocation, but have the script still 
```

## New‑post skeleton (copy/paste)

```markdown
---
layout: post
title: Descriptive Title in Title Case
date: 'YYYY-MM-DD HH:MM:SS'
image: "<prompt that can be used to generate a hero image using AI>"
description: >
  One‑sentence summary of the outcome and why it matters.
tags:
- devops
- github
---

1. TOC
{:toc}

## Why this matters

Brief context and value. What problem we’re solving and for whom.

## Prerequisites

- Tooling/versions
- Permissions/tenancy
- Assumptions

## Step 1 — <Action>

Explain briefly, then code/config.

```yml
{% raw %}
# code here
{% endraw %}
```

Notes:
- Short bullet of gotchas.

## Step 2 — <Action>

…

## Alternatives and tips

- Option A vs B
- Trade‑offs

## Conclusion

One‑sentence recap and pointer to next step.

Happy <topic>!
```

## Pre‑publish checklist

- Front matter complete (layout, title, date, tags, optional image/description).
- TOC added (`1. TOC` + `{:toc}`).
- Headings are clear; steps are sequential.
- Code blocks have language hints; YAML fenced with `{% raw %}`/`{% endraw %}` if needed.
- Links and image paths verified; captions present.
- Includes a “Notes” section for gotchas and a short conclusion with sign‑off.
