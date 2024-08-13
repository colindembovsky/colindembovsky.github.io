---
layout: post
title: 'Why Culture is so important in the Age of AI'
date: '2024-01-11 01:22:01'
image: /assets/images/2024/08/culture/ai.jpg
description: >
  GitHub Copilot is proven to improve individual productivity at the task level. However, organizations need to be intentional and systematic in how they scale GitHub Copilot broadly in order to realize organizational benefits. In this post I'll discuss why culture is so important in the Age of AI.
tags:
- ai
- process
---

1. TOC
{:toc}

> Photo by <a href="https://unsplash.com/@omilaev?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Igor Omilaev</a> on <a href="https://unsplash.com/photos/a-computer-chip-with-the-letter-a-on-top-of-it-eGGFZ5X2LnA?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
  

You've seen the marketing numbers - developers who use GitHub Copilot [complete tasks faster](https://github.blog/news-insights/research/research-quantifying-github-copilots-impact-on-developer-productivity-and-happiness/) than those who don't have GitHub Copilot. For those who have objections that Copilot doesn't provide quality code, GitHub's research shows that [GitHub Copilot Chat can improve quality](https://github.blog/news-insights/research/research-quantifying-github-copilots-impact-on-code-quality/). However, even with this data, many organizations are not seeing broad impact on their overall organizational productivity. Why is there a disconnect?

## From IDE to Production

GitHub Copilot (to be technical, Copilot Individual and Copilot Business) are focused primarily on the coding experience - users access GitHub Copilot as an IDE extension after all. At the _individual_ level, it's almost inarguable that GitHub Copilot improves speed, productivity, quality and developer happiness. But how does speed at the individual level translate to the _organizational_ level?

Think about the journey that code goes through - from the IDE to production. There may be a lot of steps such as:

- CI to build and run unit tests
- Static scanning for security issues
- Human code review
- Deployment to some test environment
- Integration testing
- Approvals from release managers
- Deployment to staging
- More approvals
- Deployment to production

Even if developers are faster _upstream_, the process between IDE and prod can dilute the gains! This is why culture is so important to realizing the benefits of AI assisted pair programming.

Let's imagine that developers gain a 20% productivity improvement with GitHub Copilot. Does that mean that deployments will be 20% more frequent? It really depends on how smooth the Code-to-Prod process is. If there is only a single senior engineer doing code reviews, they will start seeing an increase in the number of reviews (since code is being changed faster by devs). There will be more builds, so build capacity and efficiency will be impacted. How quickly can you deploy to a staging environment? What about test data management? Are manual approvals required before deploying - how efficient are those?

In other words, if you have inefficient processes, you won't realize the real impact of GitHub Copilot.

## Remember the advent of Agile?

The Age of AI reminds me a lot of the Age of Agile. Initially, many waterfall teams renamed their Product Managers as Scum Leads and called themselves Agile. Unfortunately, that didn't work. The organizations that really benefited from Agile were the organizations that really shifted their culture. Large, top-heavy groups were decomposed into smaller, autonomous teams. People were valued over process - and all of the [Agile Manifesto](https://agilemanifesto.org/) principles were truly put in place, not just paid lip service.

In the same way, organizations that truly wish to realize the benefits of Generative AI (and GitHub Copilot in particular) need to focus on culture - and the entire DevSecOps cycle. While GitHub Copilot can feel magical, it's not a silver bullet (to mix metaphors).

Bill Gates says, "Businesses will distinguish themselves by how well they use [AI]." Giving developers GitHub Copilot and expecting to see massive downstream improvements is a recipe for disappointment - you are going to have to change "business as usual" if you want to truly transform.

## Fix it where it sucks!

A common approach to continuous improvement is to "find the place where it sucks the most, and fix that." Then repeat. For example, I remember when I joined as a developer at a financial services company, the team was using Visual Studio "right-click publish" to deploy. Sometimes, the dev deploying would forget to check code into source control - meaning what got deployed wasn't always what was in source control! Also, config files frequently got overwritten. Needless to say, deployments were a pain!

To address this, we started implementing automated builds. We also instituted a rule that required us to deploy the build binary rather than deploy from Visual Studio. This meant that we were guaranteed to have the source code for the binaries deployed to production.

Once we had fixed that pain, we then created an automated release process so that we wouldn't "fat-finger" configuration files. We finally had a repeatable, reliable deployment process!

However, now that we could deploy faster and more reliably, we started to notice that we frequently had production bugs. We had no test automation, and "works on my machine" was the entirety of our QA process. Fixing the build issue highlighted the fact that we were lacking in our test automation. We started to write tests for every deployment...

You get the idea. Fixing one pain point in our process usually highlighted some other inefficiency that we had to address in order to realize the full downstream improvement.

## Realizing AI benefits requires cultural change

If you truly want to get Return on Investment (ROI) for GitHub Copilot, you're going to have to change your culture. There is no short cut. GitHub Copilot requires developers to change the way they code - which some devs embrace, and others resist. Once you've dealt with that hurdle, you'll also need to take an honest look at the rest of your process and culture - it's going to have to change!

Here are some things that you can look at improving.

### Give developers time to learn how to use GitHub Copilot

Learning a new coding language or framework takes time. It takes time to learn architecture are and industry best practices - as well as when you need to adapt them for your organization. We all know this. So why do we expect developers to instantly know how to use Generative AI? Perhaps it's the marketing and they hype - it's supposed to be intuitive after all, right? However, it takes practice to learn how to code with GitHub Copilot. It takes time to learn that you can iterate (you don't have to blindly accept the first suggestion). It takes time to figure out what GitHub Copilot is good at, and what context it needs to give good suggestions. You need to make sure you give your developers time to develop their generative AI pair-programming skills.

### Automate build, test and release

I am still shocked at how few teams have a fully automated build, test and deploy process. This should be table stakes for every application. You will never see downstream gains such as faster velocity or improved quality if you don't have these automated processes.

Implementing this is simple. Remove write-access to production environments from your developers (they may still need read access to get logs etc.). Then create a Service Principal (or similar) that has write-access and use it to automate deployments. This means that _all deployments are now automated_. This forces what is deployed to be in source control and removes manual deployment errors.

Of course, you'll have to automate infrastructure management - which is where infrastructure as code becomes so important. Invest in infrastructure as code to automate infrastructure management to ensure smooth deployment automation.

### Build out your test coverage

Even if teams do have build and release automation, test coverage remains poor in much of the industry. Once again, this should be table stakes.

Again, the implementation is simple. Don't merge PRs that don't have tests. When we started implementing this at my company, the first objection was, "We have too much code - we'd never be able to cover all of it." I then recommended that we forget about what absolute coverage is right (is it 60%? 80%? 100?) and focus instead on the _relative percentage_. Every time we deployed, we checked that coverage for this build is higher than (or at least equal to) coverage from the last build. Initially, we had 0.5% coverage (which is higher than 0%). The next deployment, we had 1% coverage, and so on. It actually didn't take long (a few months) until we had 60% coverage on most of our applications. Small, incremental changes can add up over time.

You should also start with investment in unit testing. Many teams try to "skip" unit testing and go straight to manual testing. Manual testing doesn't scale well (test automation does) and even integration testing requires test data management, which can be complex. Unit tests have the highest ROI of any testing type, so start there.

### Security Scanning

Given the increase in both the quality and frequency of cyber-attacks, security scanning, secret scanning and dependency scanning should be non-negotiable. Security scanning is non-trivial, and requires both tools and a change in culture. "Secure as we code" and "shift-left" are easy buzz words, but implementing this in real life can be challenging.

The cultural challenge for "shift-left" security requires a focus on tools and processes that are very low-friction for developers. Scanning late in the cycle on disparate tools by security teams that are incentivized to find vulnerabilities is a recipe for low remediation rates. Organizations that focus security teams on scale (setting organizational level standards) and focusing on helping developers quickly find (and easily remediate) vulnerabilities is the only way to stay ahead.

Besides static code scanning, don't forget secret scanning! You'll also need to continuously scan your dependencies to ensure that you aren't reliant on vulnerable packages.

### The GitHub Platform

Anyone who looks at GitHub will see a huge focus on AI. However, the GitHub Platform is the bedrock for achieving true productivity gains from GitHub Copilot. The platform is built around collaboration, but includes Actions (for build, test and deployment automation), GitHub Advanced Security (GHAS) for robust, low-friction, developer-friendly security and other features like Codespaces (for consistent, easy developer environments). GitHub Copilot truly shines when GitHub is the platform for your entire development process.

## Conclusion

Realizing downstream impact - through metrics like DORA and SPACE - from GitHub Copilot requires that organizations optimize their entire development process. It's not enough to give developers GitHub Copilot and expect magic - you need to focus on the entire IDE-to-production cycle, including the culture of the organization. Organizations that embrace the new Age of AI intentionally, and truly address their development processes, are going to outpace competitors and win in their markets.

Happy improving!