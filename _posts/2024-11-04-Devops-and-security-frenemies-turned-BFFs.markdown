---
layout: post
title: 'DevOps and security - Frenemies turned BFFs'
date: '2024-08-13 01:22:01'
image: assets/images/2024/2011/frenemies/frenemies.jpg
description: >
  TODO
tags:
- security
- process
---

1. TOC
{:toc}

> This post is my summary of the Universe 2024 Fireside Chat with Brian Rossi from Caterpillar Digital
  
## DevOps and security - Frenemies turned BFFs

TODO: embed Youtube URL: https://www.youtube.com/watch?v=yBlDDZhWGj4

- 250 dev teams
- wanted to create a centralized way to develop software
- we help teams to enable the SDLC
- a huge challenge was bringing security work to the dev teams
- !! how does DevOps enable cybersecurity for our org
- !! how do we get dev teams to adopt security as work?
- !! huge challenge coming from a centralized cyber team was not knowing how dev teams work
- how do we speak in terms that devs understand? They glaze over when talking about vulns
- we refer to vulns as "defects"
- !! security is part of quality
- how to devs manage quality? through issues and defects and backlogs and terms devs understand
- we changed our language to terms devs understood - then they could manage security as part of normal work
- challenge: 250 teams that own their own SDLC
- how do we ensure teams have the pieces in place to security as part of their SDLC
- started with fragmented tools: GitHub and Boards and other security tools
- how do we bring security work to where the developers live
- it's the repos! Was a huge challenge with our existing toolset
- GHAS was a natural way to get security work directly in front of the devs
- we stopped talking about vulns - it's just work that needs to be prioritized
- when do you prioritize? how long can this exist before you do something?
- DevOps org and cyber partnered very closely
- Cyber said, "YOU own your SDLC - we're not going to change it! We're bringing visibility."
- how do we make it visible? we needed metrics that devs understood to help prioritize
- how can we help them make good decisions?
- created a scorecard (letter grades) based on age of vulns
- teams were able to work towards their goal grade
- have about 15 sec engineers specifically focused on AppSec (as opposed to infra/data etc.)
- intentional about naming our org: "DevOps and Cybersecurity"
  - want devs to know we're DevOps first
  - never been a fan of the term "DevSecOps" because security isn't "special" in the life cycle
  - it's a part of what we do
  - we're there to help, not just label problems
- GHAS was important because of the integration into repos
- had to contend with the CISO that GHAS was what we needed to truly shift left
- segmented tools don't get devs the info they need early in the SDLC - which is where GHAS came in
- we see ourselves as enablers of developer teams
  - once we had that in place, the divide between dev/sec solved itself
  - devs don't feel pressure from sec
  - we don't dictate breaking builds - it's a tool that devs can use if they want
  - we can help you - but it's your decision
  - how you remediate vulns should fit into your SDLC
  - we removed the stigma of "we're here to stop you" to "we're here to help you"
  - security has been the "Office of No"
- metrics:
  - started a bug bounty program
  - 2 sets of metrics: maturity of SDLC, risk reduction
  - separated pruposefully since we don't measure risk in the SDLC if vulns are just work
  - we focused on maturing teams so that they can remediate effectively
  - !! letting vulns age is more a reflection on maturity than a reflection of risk posture
  - low scores were not a reflection of risk: they meant we needed to improve prioritization or mature our life cycle
  - gameification: self-reporting defects got you a +1, bugs found in the bounty got you a -1
  - devs felt helpless against bug bounty: have to drop everything to fix
- goal wasn't to have security drive teams to a score: it was to look at the SDLC and find a better way to do this
- if we can push code every day, I may accept more risk since I can fix tomorrow vs we deploy every 6 months, so if we find we have to fix NOW
- going faster makes you more secure
- focus on Dev Ex
  - Cyber never focused on devEx
  - if was always: Identify Risk, Communicate Risk, Address Risk
  - little opty to spend time understanding how dev teams work
  - the better your SDLC is, the more opportunity you will have for remediation
  - let devs lead: started an annual "Secure the Work" conference: a developer-led security conf (no sec pros)
  - teams with good practices that led to good outcomes got to share their stories
  - we're empowering our dev teams to be security leaders rather than just sec pros


## Conclusion


Happy securing!