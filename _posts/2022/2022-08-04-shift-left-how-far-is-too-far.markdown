---
layout: post
title: 'Shift Left - How far is too far?'
date: '2022-08-04 01:22:01'
image: /assets/images/2022/08/left.jpg
description: >
  We've all heard the mantra to "shift left" - mainly for testing but also for security. Security scanning earlier (lefter 😸) in the process makes sense, but can you shift left too far?
tags:
- security
---

1. TOC
{:toc}

> Image by [Nick Fewings](https://unsplash.com/@jannerboy62?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/left?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

I have a developer background, so App Security (AppSec) was always anathema to me. However, I had an epiphany about GitHub Advanced Security and how it is unique in it's approach - it is _security for developers_. I wrote some thoughts about that in a [previous post]({% post_url 2022-06-08-ghas-will-win-the-appsec-wars %}).

GitHub Advanced Security (GHAS) allows you to reduce risk _without impeding velocity_. This is a big deal in today's fast-paced world. The way that GHAS does this is by centering AppSec on the developer, while still meeting requirements of security professionals. Integrating AppSec into the developers' daily workflow with very low friction is the secret to securing your software effectively.

GHAS centers itself around the _repo_ and the _Pull Request_. I have had a number of customers ask why GHAS does not have an IDE plugin. If shifting left is the Holy Grail of AppSec, and GHAS is built to be developer-centric, then why isn't GHAS in the IDE? Isn't that the furthest left we can shift?

Or would that be too far left?

## How Far Left is Too Far?

Let's take a moment to consider where in the life cycle various GHAS features work:

Feature|Phase
--|--
Secret Scanning|After pushes to the repo. If you have [Push Protection](https://docs.github.com/en/enterprise-cloud@latest/code-security/secret-scanning/protecting-pushes-with-secret-scanning) enabled, secrets are scanned before the push.
Dependency Scanning (SCA)|After pushes to the repo and in PRs via [Dependency Review](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-dependency-review).
Code Scanning (CodeQL)|During builds and surfaced in PRs.

It seems that Push Protection is the only feature that occurs before a `push` to the repo. Dependency scanning and code scanning are centered around the repo or PR. Why is the PR the center of GHAS, rather than the IDE? Wouldn't it be even faster if the IDE could surface vulnerable dependencies and vulnerable code before developers push changes to the repo?

### IDEs

Developers can be picky about their IDEs. While many modern IDEs are extensible, there is no standard for IDE extensibility. This means that any policy enforcement at the IDE is near impossible, since you'd have to implement that policy for all IDEs. You could mandate a single IDE, but that doesn't always work.

Additionally, there's no simple way to force developers to turn certain tools and plugins on in the IDE. Any process that relies on the IDE is relying on the developer to remember to turn on the tool. And what about shared configuration? Relying on configuration files may work - but many IDEs store preferences on the workstation in personal folders rather than in repos, so sharing common config can also be a challenge.

IDEs are great for "simple" analysis - linters that enforce coding standards work really well in IDEs, assuming you can effectively share the linting rules. Most linters are built this way, storing configuration dotfiles alongside the code. Most linters are _fast_ because they typically require very little compute, so running them in the IDE doesn't distract the developer.

However, most security analysis tools (worth their salt) tend to require heavier compute and take longer to scan because of the more complex problem domain. Putting code scanning into an IDE becomes a resource hog for developers (have you ever seen a developer waiting for an IDE to compile their code - it's not pretty!). Furthermore, inundating developers with tons of results can be distracting and end up reducing the remediation effort of the developer since they get fatigued by noisy alerts.

### Baked in or optional?

Security testing that isn't built into the inner sanctum of your code is _effectively optional_. External tools require someone to build them, install and configure and maintain them, integrate them and automate them. Even if you buy a 3rd party tool rather than build it yourself, you still have to operate, configure, intergrate and automate it yourself. This friction and extra overhead tends to cause developers to avoid these tools - and you lose any value they offer if just one person "forgets" to run the scan.

### Background analysis

What about running the code scanning _in the background_ on the developer laptop? This can get problematic because of compute constraints, and may end up with the situation where code is changed before the scans complete, so you get alerts for code that has already changed or been removed - way too much friction and frustration.

### CLI Tools before pushing code

You could require developers to run CLI tools before pushing code - but this is now outside of the IDE anyway. Developers will invariably forget to run the tool, or just avoid running it since it is disruptive to their coding workflow.

### Pre-commit hooks

What about pre-commit hooks - what about running code scanning there? Once again, typical code scanning takes in the order of minutes - far too long for a pre-commit hook. Developers would have a fit if it took 10 minutes to scan the code before a successful push! Heck, even 1 minute is too long to wait for a 
push to succeed.

### Data for dependency scans

Dependency scanning (SCA) is performed on the repo with GHAS. While the dependency graph could be built in the IDE, how would the IDE compare the dependency graph to CVE/CWE databases to determine if any package contains a vulnerability? Either the IDE would have to download the databases or make API calls, which could be too slow and disrupt the daily developer workflow.

## The sweet spot

Taking the above considerations into account, it becomes clear that placing security scanning at the repo/PR is as far left as you should go. Not only does this make security remediation a _team sport_ since team members can collaborate around alerts/remediation process, but this is very little disruption to the daily workflow of a developer. For complex codebases where scanning takes longer than 10 minutes and could potentially slow CI/CD, scheduled jobs or parallel workflows (a CI workflow and a scanning workflow) are perfectly acceptable workarounds.

Developers are already used to collaborating around the PR. The PR is already the rallying point for code review, automated unit testing, linting and other quality gates. GHAS allows teams to add security testing into this pivot point smoothly. This means developers can keep using whatever IDEs they want - but still gain all the benefits of security scanning early and often in the software life cycle.

Dependabot runs post-push (and on a schedule) on the repo and is able to then compare the dependency graph to the vulnerability databases. Automated PRs to bump to patched versions further aids developers to quickly and easily remediate vulnerable packages with very low friction and interruption.

Secret scanning is the one exception - that you want to shift as far left as possible to prevent secrets from ever making their way into the shared repo. Secret Scanning in GHAS scans a repo's entire history when you enable it for the first time, but you can also turn on Push Protection to ensure that secrets are kept out of the repo in the first place! Under the hood this is achieved conceptually by a pre-commit hook - but the computation time for secret scanning is far smaller than that required to perform code analysis. Secret scanning tends to complete well within seconds, allowing it to be shifted "more left" to the `push`.

# Conclusion

Shifting left is critical for AppSec in today's world - but you can actually shift too far left. GitHub Advanced Security shifts as far left as possible, but not into the IDE. This decision is deliberate and considered, since IDEs are not ideal for code and dependency scanning. Push protection ensures that secrets don't enter the repo, but Dependency scanning and Code scanning are centered on the repo and PR where there is little friction for the development inner loop and encouragement of collaboration to remediate security alerts.

Happy securing!
