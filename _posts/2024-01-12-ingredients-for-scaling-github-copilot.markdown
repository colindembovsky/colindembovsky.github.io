---
layout: post
title: 'Ingredients for scaling GitHub Copilot'
date: '2024-01-11 01:22:01'
image: /assets/images/2024/01/scaling/developing.jpg
description: >
  GitHub Copilot is proven to improve individual productivity at the task level. However, organizations need to be intentional and systematic in how they scale GitHub Copilot broadly in order to realize organizational benefits. In this post I'll discuss some considerations for scaling GitHub Copilot.
tags:
- ai
- process
---

1. TOC
{:toc}

> Photo by <a href="https://unsplash.com/@timmykp?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Tim van der Kuip</a> on <a href="https://unsplash.com/photos/man-sitting-on-chair-wearing-gray-crew-neck-long-sleeved-shirt-using-apple-magic-keyboard-CPs2X8JYmS8?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>

I work with a number of enterprises with large development communities: 5,000 - 25,000 developers. Managing DevSecOps at this scale is challenging, and keeping up with the pace on innovation in today's AI-eaten world only adds complexity. While most organizations have dipped their toes into the generative AI waters, many are struggling to realize broad organizational benefits.

# Speed for the Individual

Most customers I work with would agree (even if only intuitively) that GitHub Copilot is a productivity booster for developers. However, executives are often skeptical when they see numbers such as [developers who use GitHub Copilot complete tasks ~55% faster than developers without GitHub Copilot](https://github.blog/2022-09-07-research-quantifying-github-copilots-impact-on-developer-productivity-and-happiness/). This is not just vaporware from GitHub - customers are also reporting large productivity gains:

1. Duolingo is seeing a [25% increase in developer speed with GitHub Copilot](https://github.com/customer-stories/duolingo)
1. Coyote Logistics is reporting [50% decrease in time to write Terraform config files](https://github.com/customer-stories/coyote-logistics)
1. Marcado Libre reports a [50% reduction in time spent writing code with GitHub Copilot](https://github.com/customer-stories/mercado-libre)

These are just a few examples - there are more stories reporting similar numbers.

We may argue over _exactly_ how much improvement developers get (and for what use-cases) - but there is enough evidence to assert that GitHub Copilot _makes individuals faster_. But how do we scale this individual productivity gain to the organization?

# Ingredients for Scaling

There are some patterns that we see when analyzing customers that are successful:

* Executive mandate
* Systematic approach
* Allowing time for developers to learn how to code with GitHub Copilot
* Super simple onboarding
* Establishment of Communities of Practice and identification of Champions
* Tying GitHub Copilot to initiatives
* Pragmatic measurement and measuring the right things

## Executive Mandate

It is imperative the there is an executive mandate to use GitHub Copilot. Given the evidence of how effective GitHub Copilot is, executives should be tasking their teams with using GitHub Copilot and learning how to benefit from it - if for nothing else than to stay ahead of competitors!

## Systematic Approach

"Just turn it on" is not a good rollout strategy. This applies to _any_ tool - not just GitHub Copilot. Organizations must consider how they are going to scale out. Team-by-team is a common strategy. Other strategies include "lighthouse teams first" or "language by language" or some other means of starting small and expanding out. Starting with developers that are _hungry_ for GitHub Copilot is crucial - these folks are more likely to spend the time it takes to become good at GitHub Copilot, iron out networking challenges and other onboarding road bumps. Once these teams have gained some experience, they become key to scaling out GitHub Copilot skills to other developers and teams.

## Allowing Time

GitHub Copilot can feel magical - but it is certainly not infallible. It takes time for developers to learn how to craft prompts so that they are useful, what the limits of GitHub Copilot are and how to change the way they code to fit GitHub Copilot in. This would be the same for developers adopting Test Driven Development (TDD) or eXtreme Programming (XP) - new ways of coding take time to learn and to adapt to.

Many developers try a couple of (not so good) prompts and conclude that GitHub Copilot "isn't that useful." However, if given time and examples, most developers start to learn how to better craft prompts and to learn the boundaries of GitHub Copilot's capabilities. Giving up too soon (or piloting too quickly) will prevent successful scale out.

## Super Simple Onboarding

GitHub Copilot seats are "pay as you use". This is different to GitHub Enterprise or GitHub Advanced Security licenses that are purchased up-front. This gives customers much more flexibility in how and when seats are assigned. While giving every developer access to GitHub Copilot from day 1 may be easy, it is not optimal. If customers are not going to give everyone access, they have to think about how they are going to manage how and when developers get seats. Making this process super simple is key. A few of my customers require developers to fill out a form in their internal ticketing system which in turn calls an API to allocate a GitHub Copilot seat without the need for an approval. They have effectively made seat allocation self-serve.

Along with self-serve onboarding, customers must create a centralized knowledge base with onboarding docs, starter docs and demos. Many enterprises have proxies or other networking and firewall rules that prevent GitHub Copilot from working out of the box. Having documentation about how to configure proxies and how to authenticate GitHub Copilot is very important. Along with that, some docs that show how to get started (sample prompts, sample use-cases etc.) and demo videos are critical for success.

## Establishment of Communities of Practice and identification of Champions

GitHub Copilot is a tool that requires continuous investment - since it is an art as well as a science, developers need to continue to develop their prompt crafting skills. Additionally, GitHub Copilot is continuously improving and new features are being added frequently. The best way to support a skill that needs continuous investment is a Community of Practice (COP) (or Center of Excellence or Guild or whatever you call this cross-cutting construct within your organization). This CoP needs to meet frequently and continuously evangelize tips and tricks and wins to keep momentum high.

Along with the CoPs, scaling requires identifying Champions - these are super-users, tech leaders and influencers within your organization's development community. These folks need to be recognized and empowered to become GitHub Copilot Advocates internally. The more of these you build, the faster you will scale. The Champions are going to be those that are excited about GitHub Copilot, but also those that make the most GitHub Copilot requests (and have the highest acceptance rate). Identifying Champions by language is also helpful.

While GitHub does provide expert services and there are many GitHub Partners that can assist organizations to scale out GitHub Copilot, organizations must develop their own internal competency and programs in order to create sustainability.

## Tying GitHub Copilot to initiatives

Most developers don't use tools for the sake of tools - they tend to look for the best tool for the job. Most organizations have existing initiatives for their development teams - improving velocity, app modernization, reducing technical or security debt and increasing test coverage are examples. When developers _have something to tie learning GitHub Copilot to_ they are more willing to invest time and effort. This is going to accelerate and widen adoption.

## Pragmatic measurement and measuring the right things

Along with tying GitHub Copilot to initiatives comes pragmatic measurement, as well as measuring the right things. If you tie GitHub Copilot to an initiative to improve test coverage, then you probably won't (initially) see an improvement in velocity. Being pragmatic (and targeted) with your metrics will lead to faster realization of value - not simply because of gamification, but because you will be measuring the right things. Improving maturity in what is measured (and how those measurements are interpreted and applied) is a requirement for success at scale.

# Conclusion

Enterprises must be systematic about their approach to GitHub Copilot. Enterprise that don't invest in mastering AI-assisted pair programming and generative AI in DevSecOps are going to fall behind. While these tools are novel today, they are rapidly becoming table stakes. Enterprises must be intentional about this technology - just as they should be intentional about adopting any technology. By applying the ingredients I've outlined above, enterprises can confidently scale GitHub Copilot - and realize organizational improvement faster and more sustainably.