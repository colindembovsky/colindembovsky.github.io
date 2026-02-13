---
layout: post
title: 'GHAS Will Win the AppSec Wars'
date: '2022-06-08 01:22:01'
image: /assets/images/2022/06/fly-d-9PivUW7l1m4-unsplash.jpg
description: >
  GitHub Advanced Security is positioned to win the "AppSec Wars". In this post I go over why I think this is the case.
tags:
- security
- github
---

1. TOC
{:toc}

> Image by [FLY:D](https://unsplash.com/@flyd2069?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/cyber-attack?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

> Special thanks to [Derek Chowaniec](https://www.linkedin.com/in/derekchowaniec/) who helped me "see the light" and had some input on this post!

In my first few years as a developer, I used Team Foundation Server's [Team Foundation Version Control (TFVC)](https://docs.microsoft.com/en-us/azure/devops/repos/tfvc/what-is-tfvc?view=azure-devops) which is a centralized source control system similar to [SubVersion](https://subversion.apache.org/). I remember being introduced to Git and thinking, "Who would want to use a source control system where you can rewrite history!?" But all the cool kids were using it, and I eventually "came to the light" and now I am a huge proponent of Git. In fact, I have presented a talk a couple of times at VSLive in which I posit that you cannot _effectively_ do modern software development without Git.

For a brief period, there was a war between Git and Mercurial and Git won. We see a similar war at the moment between [GitHub Advanced Security (GHAS)](https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security) and other entrenched security products such as [Fortify](https://www.microfocus.com/en-us/cyberres/application-security), [Checkmarx](https://checkmarx.com/), [Snyk](https://snyk.io/) and [SonarQube](https://www.sonarqube.org/). Gazing into my crystal ball, I predict that GHAS will win the war - at least, where it's competing: Software Composition Analysis (SCA), Static Application Security Testing (SAST) and secret scanning.

I spent many years using Team Foundation Server - which later became Visual Studio Team Services before being re-branded again as [Azure DevOps](https://azure.microsoft.com/en-us/services/devops/). However, when [Microsoft purchased GitHub](https://news.microsoft.com/announcement/microsoft-acquires-github/) for $7.5B, I reluctantly started to invest in learning GitHub, but soon was advocating vociferously for GitHub. As a DevOps practice lead at a large Microsoft partner organization, I formulated service offerings like a GitHub QuickStart and a GitHub Advanced Security (GHAS) Adoption offering. Eventually, I joined GitHub and have been here since September 2021.

I am a developer at heart - a code-slinger. I don't get to code as much as I used to because I spent 10 years helping developers develop better software faster - but I have always identified more with developers than ops and infrastructure engineers. I have also never been deeply interested in _security_.

Now even though I formulated (and delivered) GHAS Adoption even before joining GitHub, it wasn't until I was actually tasked with selling GHAS to customers that I understood just how unique GHAS is. My experience with security was almost always as a "necessary evil" - something that was grudgingly necessary but was always a nuisance.

Of course, security is increasingly important and cyber attacks are increasingly sophisticated, so this "nuisance" is no longer something that organizations can ignore or treat casually! So why did I have such a visceral and negative reaction to security as a developer?

## What's Wrong with AppSec

Application Security (AppSec) has been around for the better part of two decades. [OWASP](https://owasp.org/) was formed in 2001 and developers have all had to endure security training of some sort during that same time period. We've also had plenty of tools in this space - SonarQube, Fortify, VeraCode, Checkmarx, Snyk and others. And yet applications are no more secure today than they have ever been. Something in the AppSec space is failing.

After thinking about why AppSec is failing, I have some thoughts.

### Security Tools are made for Security Pros

Security tools are created for security professionals, not developers. This may seem obvious, but it has some far-reaching consequences.

Firstly, if developers are going to send their code (or binaries) off to some security team with their own security tools for security scanning, they subconsciously (or sometimes consciously) stop taking responsibility for writing secure code. After all, if the scanning tool is something I can't use or understand and some other team is going to run it, why should I bother thinking about it while I'm coding now?

There was a similar issue when Dev and Ops used to be separate, before the popularity of DevOps. Developers who never had to operate the code never thought about how to write code that behaved well (or was even instrumented) in production - since "those ops people will do that." DevOps has shown that a live-site culture in your developers is critical for modern software development.

### Security is often too little, too late

As hard as it is to find good developers, it's even harder to find good security professionals. Some studies show that there is only 1 security pro for every 800 developers! Because of this, security professionals are often overwhelmed, and become a bottleneck. Any time you have queues and bottlenecks, you end up with longer lead times.

Furthermore, scanning is typically performed late in the development lifecycle where results are sometimes ignored because it's too late to do anything about them. Organizations often only fix critical errors because they don't have time to fix the merely high or medium severity errors. This leads to risk.

### Results are too noisy

I remember seeing the results of my first SonarQube scan on a simple repo. It was noisy. There were hundreds of findings, and I didn't know where to start or how reliable the findings were. Eventually, it became too hard to sift out false positives, so we just ignored the findings. This seems to be a common experience for more developers on most security tools.

### Security is expensive

Security tooling is often billed per scan or per app. This leads to companies categoriing applications so that they only scan their critical or important applications. However, hackers don't care if you have a "small app that almost no-one uses" - that could be the very vector they use to breach your systems. In other words, if you're not scanning all your apps, you're at risk. But some security tools make this prohibitively expensive. Some companies scan all their apps, but not continually because it is too expensive. Again, this leads to risk, since new vulnerabilities are being discovered all the time, so you need continuous scanning to be secure.

## Quantum Leaps in Developer Productivity

My first job was writing C/C++ code with [CORBA](https://corba.org/) on Linux. We used a basic text editor for writing code, since there really wasn't a good IDE at that time. Debugging applications involved writing a ton of `printf()` statements, running the application and watching the console. It was painful, to say the least.

My second job was at a Microsoft shop. I was introduced to C# and to Visual Studio. I remember the first time I set a breakpoint and hit F5. It was like magic! If you've never had to develop without being able to set breakpoints, you probably don't understand what a huge leap in productivity interactive debugging is!

Similarly, in my first job, we had about 8 developers watch the build on Friday afternoons. We had some scripts, but the build process was not automated. We would throw some meat on the grill and crowd around the designated build master for that week, and woe to the developer who's code broke the build!

I remember when I set up my first automated build (using MSBuild) in Team Foundation Server. It even queued automatically when I check in my code!

Both interactive debugging and automated builds are taken for granted by most developers today - but we forget what quantum leaps in productivity they were!

I think GHAS is the quantum leap we've been waiting for in AppSec.

## What Sets GHAS Apart

I am only now beginning to be able to articulate why GHAS is so different. I am starting to have some language for something that was very _intuitive_. I only now am realizing that GHAS felt intuitive to me because I am a developer - and GHAS is made for developers!

### GHAS is made for Developers

GHAS is not a separate tool that requires a context switch for developers. It is baked into GitHub - into the platform where developers spend their time. It integrates into Pull Requests very naturally and so it is very low friction for developers. This encourages developers to become more security conscious without interrupting or hindering flow.

### GHAS is baked into the platform

Most security tools require some compute somewhere - Fortify runs on a VM, for example. This means someone has to manage and operate the scanning infrastructure - compute, memory, disk space, networking, configuration, patching and upgrades. GHAS is built into GitHub and requires no additional management overhead than just GitHub.

This is one of the reasons why GitHub decided not to "shift left" into the IDE - developers may run vastly different IDEs for local development, but the team revolves around the PR and the CI build - and that's where GHAS "lives".

### GHAS is automated

GHAS is fully automated, so it requires very little thought once it is configured. This also enables continuous scanning, again without hampering flow. Automation also means that it can be scaled quickly.

### GHAS is scalable

GHAS is the only security tool that has any sort of "instant on" capability. Both secret scanning and dependency scanning (SCA) can be turned on across all your repos instantly. SAST through CodeQL relies on a build, so it can't (yet) be turned on instantly, but if you have mature builds, it can be integrated easily. This means the time between acquiring GHAS and getting results (time to value) is far shorter than other security tools.

### GHAS has a unique pricing model

GHAS is charged according to "active committers". Any developer that has committed code to a GHAS-enabled repo in the last 90 days consumes a license. If that same developer pushes to 50 different repos, still only 1 license is consumed. This means that organizations have a predictable pricing model (number of developers multiplied by the GHAS license cost) and they can scan _as many apps as they want as many times as they want_.

### GHAS has very little noise

GHAS is designed to be very "clean" - that is, it has extremely low false positive rates (I won't get into why in this post). This means that findings are arguably more actionable, and many customers that are using GHAS are seeing decreasing mean time to remediate (MTTR) when using GHAS. Developers also trust the findings more when there are fewer false positives.

### GHAS is extensible

GHAS is powerful in its own right, but GitHub knows it can't do everything. For example, GHAS does not (at this point) have a container image scanning solution. However, because our SAST offering is built using SARIF files, _any security tool that can produce a SARIF file can be integrated into GHAS_. This allows you to quickly integrate scanning tools like Anchore and Trivy into GHAS to give the developer a consistent experience. Even some Dynamic Application Security Testing (DAST) tools like StackHawk have [extensions that upload SARIF files](https://github.com/marketplace/actions/stackhawk-hawkscan-action#codescanningalerts).

### GHAS leverages the Global Security Community

CodeQL queries are open source - you can see them in the repo [here](https://github.com/github/codeql) under the language folders. These standard query suites are built by GitHub as well as security professionals across the globe. GitHub maintains the repo to ensure high quality. The point is that you don't have to know anything about CodeQL to run comprehensive scans - ensuring that you can get going quickly. Of course, if you're fortunate enough as a developer to have security professionals in your company, they can create custom queries since CodeQL is a T-SQL-like language. Hopefully, they'll contribute back to the CodeQL repo!

Because CodeQL encourages collaboration, we all get to benefit from the _collective brain power of the security community_. Just as Actions is by far the most widely used CI/CD tool on the planet, in large part because of the community, CodeQL is poised to becoming the most widely used SAST tool on the planet. This means that the standard query suites are growing all the time, adding further value _which you don't pay more for_.

## Conclusion

GHAS is the new kid on the block when it comes to AppSec. I think it is poised to win the AppSec wars because of the reasons mentioned above, but especially because it is _focussed on developers and developer experience_. If you're wanting to grow in your AppSec journey, you need to get onto GHAS!

Happy securing!
