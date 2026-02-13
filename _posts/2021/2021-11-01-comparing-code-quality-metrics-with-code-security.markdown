---
layout: post
title: Comparing Code Quality Metrics with Code Security
date: '2021-11-01 01:22:01'
image: /assets/images/2021/11/code-quality/matrix.jpeg
description: >
  Code security is becoming more important for modern software development. What about code quality metrics? How do code quality metrics and code security compare and contrast? I'll discuss some thoughts in this post.
tags:
- security
---

1. TOC
{:toc}

# Code Security

Code security has traditionally been an "after the fact" activity. Developers would develop, build, test applications, and then when they're ready to ship to production, attempt to get a security sign-off. This not only isolates developers from security professionals, but this usually ends up either blocking deployments completely or causing teams to deploy vulnerable code with the promise to come back and fix later.

The irony is that we've had security awareness, training and tooling for decades. [OWASP](https://owasp.org/) was founded 20 years ago! Tools like Black Duck (2002), Fortify (2003), Veracode (2006) and Checkmarx (2006) are in a rich landscape of security tools. So why are we still seeing so many breaches?

> Note: Even Semmle (which was acquired by GitHub and turned into CodeQL) has been around for many years.

"Not in my code! Vulnerabilities are in infra," I hear you state confidently. But the Verizon Data Breach Investigation reports between 2016 and 2020 show that the primary attach vector in breaches is _application flaws_. Furthermore, GitHub's Data Science team analyzed 70 million lines of open source code and showed a linear relationship between lines of code and security threats introduced. In other words, the more code you have, the more potential threats you have.

Many of these companies have been banging the "shift-left" drum: that is, integrate security earlier into the development life cylce. Still we don't see drastically more secure code. Why?

I believe the primary security failure in the industry at the moment is due to the fact that most security tools are build by security professionals for security professionals.

# Security for Developers

[GitHub Advanced Security](https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security) (or GHAS) is a game-changer. GHAS is security _focused on developers_. By integrating security tooling into the very platform and into the daily workflows developers use, GHAS finally empowers developers to be responsible for security in a natural way.

I won't go into all the features of GHAS in this post, but I want to focus on a key component of GHAS and that is integrating in SAST (Static Application Security Testing) into CI/CD using [CodeQL](https://docs.github.com/en/code-security/code-scanning/automatically-scanning-your-code-for-vulnerabilities-and-errors/about-code-scanning-with-codeql) within [GitHub Actions](https://docs.github.com/en/actions), the native GitHub automation engine.

The beauty of CodeQL is that you can integrate it into your CI/CD workflow _without having to write CodeQL queries_. Of course, if you have security professionals on your team or in your organization, they can write custom queries. But even if you don't, you can tap into the growing suite of [standard queries](https://github.com/orgs/codeql/packages) that is constantly being updated by the security community.

> Note: CodeQL can also be [configured](https://docs.github.com/en/code-security/code-scanning/automatically-scanning-your-code-for-vulnerabilities-and-errors/configuring-code-scanning#using-queries-in-ql-packs) to run maintainability and reliability queries by using the `security-and-quality` suite instead of the default `security-extended` suite.

# Code Quality Metrics

Many teams try to measure quality through code quality metrics, and there are tools that are good at collecting these metrics - like [SonarQube](https://www.sonarqube.org/). Using SonarQube you can get a grade on how maintainable your code is for example.

Sounds great - we should all be deploying high quality code, right?

## Can You Trust Code Quality Metrics?

There are some problems with code quality metrics. A common code quality metric is _cyclomatic complexity_ - a measure of how many paths there are through a portion of code. Perhaps we want to ensure that no single file has a cyclomatic complexity higher than 10. Now if a file has a cyclomatic complexity of 11, we _might_ have a "bad file" - or maybe the logic is just complicated.

In the "negative" direction, we may or may not agree with the metric result. What about the "positive" direction? If a file has a cyclomatic complexity of 7, does that tell us if the code is good or not?

You begin to see the problem - if we can't trust the metrics, then what value do they really have? If we have code that "scores high" in code quality metrics, can we conclude definitively that we have good code?

In contrast, assuming we have a good security tool like CodeQL that is known to have very low false positive rates, we can most definitely trust the code security alerts. If we run through the CodeQL suite and there are no alerts, we have high confidence in the security of our code!

Another problem is where (or when) in the lifecycle you really care about code quality metrics. Let me explain it using a hypothetical scenario.

# When Do You Care - A Thought Exercise
Imagine your team is maintaining an application that's in production and has a solid user base. Let's imagine it's an e-commerce site, something like Amazon. Now imagine that you are implementing improvements to the checkout experience to ensure that customers can more easily pay using PayPal. The team has been working on the "PayPal Improvement Feature" for several weeks and are getting ready to deploy. Black Friday is coming, and you know that it's a huge day for your company and site because of all the specials that you run. Your team has been unit testing and they've been running continuous deployment to staging environments and they've demonstrated performance is acceptable through integration testing. All systems are a go!

But suddenly you get a B for some code quality metric. Perhaps the team have been getting A's so far - but the latest merged code has some code that could be written in an academically better way. Mind you, no tests are failing, and integration and performance testing are all green. What do you do? Are you going to deploy? Or block the deployment until the team has improved the code quality metric from a B to an A?

Let's now imagine that your security scans reveal that there is a vulnerability in the latest merged code. Do you still deploy, or get the team to fix the vulnerability?

# Quality vs Code Quality Metrics

I'm not for a second insinuating that _quality_ is unimportant. What I'm saying is that, in general, there are diminishing returns on measuring code quality _metrics_. We really conflate code quality and code quality metrics, but they are different things.

Customers that use your code don't care about your code quality metrics. _Performance, reliability, how fast you release new features_ - these are the things that your customers really care about (product quality if you will). The question is how many of these are predicted by code quality metrics? In other words, can we definitively say that code with high quality metrics is always performant? Or scalable? Or secure? On the other hand, if our site has good (or even good enough) performance, how much should we care about code quality metrics?

## Quality Gates

I'll repeat: I'm not saying that _quality_ is unimportant. I think that there are other far more effective ways to measure and ensure quality than quality code metrics. 

I have coached many teams that are looking to implement testing that start by attempting to implementing UI testing. After all, they reason, if the test is at the UI layer, then we can ensure quality through the service layer to the data layer - no need to test those separately, right? 

It turns out that this is a trap: different types of testing have different challenges, and differing rates of Return on Investment (ROI). I wrote about this [here]({% post_url 2013-07-18-why-you-absolutely-need-to-unit-test %}). Unit tests have a high ROI, since they are usually easy to write and don't require data or environment management. Integration and Functional tests are more expensive to write and maintain, since you need consistent, stable environments and have to manage test data. UI tests are notoriously fragile. ROI diminishes quickly beyond unit testing.

In the same manner, the ROI for code quality metrics diminishes over time. Teams can (and should) implement _quality gates_ to ensure that deployed code meets quality criteria. Assuming you peer-review Pull Requests, implement unit testing, have some Integration and Functional tests, and monitor performance of your code running in production, what real value do code quality metrics add? It definitely has _some_ usefulness, but I would argue over time its usefulness diminishes over time, especially if you have other quality gates in place.

## Code Quality Metrics Over Time

Let me suggest a rough graph showing criticality of Code Quality Metrics over time:

![Criticality of Code Quality Metrics Over Time](/assets/images/2021/11/code-quality/code-quality-chart.png){: .center-image }

Criticality of Code Quality Metrics over time.
{:.figcaption}

Any good team is going to enforce quality through mechanisms like [protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/about-protected-branches), peer code-review, unit tests, automated build and deploy workflows and integration and performance testing. Code quality metrics beyond these will arguably have value before initial deployment, and then taper in criticality over time.

## Code Security Over Time

In contrast, here's how I think a rough graph of criticality of Code Security looks over time:

![Criticality of Code Security Over Time](/assets/images/2021/11/code-quality/code-security-chart.png){: .center-image }

Criticality of Code Security over time.
{:.figcaption}

If we assume that your code base is going to grow, and that attackers are going to uncover more vulnerabilities in your dependencies and come up with new attack vectors, we can assume that threats are going to increase over time! So the criticality of code security is going to keep increasing over time. You (or the community) may uncover vulnerabilities in dependencies tomorrow that are thought to be safe today. Constant vigilance is required.

## Consequences

We can also contrast the _consequences_ of code quality metrics and code security being ignored. Going back to our Thought Exercise, you may well decide to deploy code that has a B rating for some Code Quality Metric. Let's imagine that this causes the PayPal checkout experience to demonstrate some allowable performance impact (like taking .75 seconds instead of .5 seconds to complete). If your quality gate for performance is .8 seconds, you're still within your performance quality gate, so while you probably do want to fix this at some stage, but the consequences of ignoring this metric are minimal.

Let's assume no-one is going to ignore security vulnerabilities that are surfaced through tooling. More likely, teams are not going to be performing code security scanning regularly. But what are the consequences of not ensuring that the PayPal checkout experience is secure? What would the impact be if a customer account is hacked because of a flaw in your code?

Clearly, the risk presented to not measuring code quality metrics and the risk of not securing code orders of magnitude apart.

# Conclusion

Code Quality Metrics are useful, but their criticality typically decreases over time, especially when teams implement good quality gates in their software development life cycle. The criticality of Code Security, on the other hand, steadily increases over time as code bases and attack vectors grow.

While there is a lot of tooling in both the Code Quality Metrics and Code Security spaces, GitHub Advanced Security offers a unique platform that enables developer-first security, integrating security into developer workflows naturally and seamlessly, making it an indispensable tool for modern software development.

Happy securing!