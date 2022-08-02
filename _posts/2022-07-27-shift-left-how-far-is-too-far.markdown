---
layout: post
title: 'Shift Left - How far is too far?'
date: '2022-06-21 01:22:01'
image: /assets/images/2022/06/safe.jpg
description: >
  We've all heard the mantra to "shift left" - mainly for testing but also for security. Security scanning earlier (lefter 😸) in the process makes sense, but can you shift left too far?
tags:
- security
---

1. TOC
{:toc}

> Image by [olieman.eth](https://unsplash.com/@moneyphotos?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/safe?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

I have a developer background, so App Security (AppSec) was anathema to me. However, I had an epiphany about GitHub Advanced Security and how it is unique in it's approach - it is _security for developers_. I wrote some thoughts about that in a [previous post]({% post_url 2022-06-08-ghas-will-win-the-appsec-wars %}). "Shift left" is mantra that's become so overused that its almost lost its meaning.

GitHub Advanced Security (GHAS) allows you to reduce risk _without impeding velocity_. This is a big deal in today's fast-paced world. The way that GHAS does this is by centering AppSec on the developer, rather than the security professional. Integrating AppSec into the developers' daily workflow with very low friction is the secret to actually securing your software effectively.

If shifting left is the Holy Grail of AppSec, and GHAS is built to be developer-centric, then why is GHAS not present in the IDE? Isn't that the furthest left we can shift? Or would that be too far left?

## How Far Left is Too Far?

Let's take a moment to consider where in the life cycle various GHAS features work:

Feature|Phase
--|--
Secret Scanning|After pushes to the repo. If you have [Push Protection](https://docs.github.com/en/enterprise-cloud@latest/code-security/secret-scanning/protecting-pushes-with-secret-scanning) enabled, secrets are scanned before the push.
Dependency Scanning (SCA)|After pushes to the repo and in PRs via [Dependency Review](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-dependency-review).
Code Scanning (CodeQL)|During builds and surfaced in PRs.

It seems that Push Protection is the only feature that occurs before a `push` to the repo. Dependency scanning and code scanning are centered around the PR. Why is the PR the center of GHAS, rather than the IDE? Wouldn't it be even faster if the IDE could surface vulnerable dependencies and vulnerable code before developers push changes to the repo?


