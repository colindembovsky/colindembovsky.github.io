---
layout: post
title: 'DevOps and security - Frenemies turned BFFs'
date: '2024-10-31 01:22:01'
image: /assets/images/2024/2011/frenemies/frenemies.jpg
description: >
  This post is my summary of the Universe 2024 Fireside Chat with Brian Rossi from Caterpillar Digital.
tags:
- security
- process
---

1. TOC
{:toc}

# DevOps and security - Frenemies turned BFFs

This post is a summary of some of the key insights from a recent GitHub Universe 2024 Fireside Chat with [Brian Rossi](https://www.linkedin.com/in/brian-rossi-61b840b/), Director, DevOps and Cybersecurity at Caterpillar.

<div style="text-align: center;">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/yBlDDZhWGj4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

## Centralized Approach and Team Autonomy

Creating a secure culture at scale requires a centralized _approach_ (not a centralized _team_). The centralized approch can streamline processes, ensure a uniform standard of security, and simplify the adoption of security measures within the Software Development Life Cycle (SDLC). However, a key challenge in this approach is balancing the centralized vision with the autonomy of various development teams. For organizations managing many teams — each with distinct workflows — adopting a flexible, support-oriented strategy is crucial to avoid being perceived as imposing rigid controls on SDLC ownership.

## Enabling DevOps Rather Than Policing Security

Integrating security into the SDLC through DevOps practices rather than being "The Office of No" fosters a collaborative environment where security is seen as part of quality rather than a separate function. This approach allows security to naturally align with the processes that development teams are already using to manage quality, such as tracking issues, defects, and backlogs. By embedding security into these familiar workflows, security becomes “just another type of work” that developers prioritize based on risk, urgency, and capacity, enhancing both the security and efficiency of software development.

## Language and Terminology Matters: From Vulnerabilities to Defects

One of the most effective strategies in scaling application security is to adapt terminology that resonates with development teams. Reframing vulnerabilities as “defects” places security issues in a context developers understand, enabling them to treat security as a quality issue rather than an external mandate. This subtle shift in language helps bridge the gap between security and development, making it easier for developers to prioritize and track security issues in line with other quality concerns.

## Distributed Ownership of Security Across Development Teams

In organizations with numerous development teams, each owning its SDLC, enabling effective security practices can be daunting. This decentralized ownership requires a scalable approach, where security teams provide visibility and guidance without mandating a one-size-fits-all approach. By emphasizing visibility and equipping teams with security metrics tailored to their development processes, cybersecurity becomes an enabler rather than a roadblock, allowing teams to prioritize security work autonomously while maintaining a consistent standard across the organization.

## Leveraging GitHub Advanced Security (GHAS) for Seamless Integration

GitHub Advanced Security (GHAS) offers an integrated solution that brings security work directly into the developer workflow. GHAS provides insights at the repository level, where developers are already managing their code, reducing the friction associated with fragmented security tools. By surfacing security issues within the repository, organizations ensure that security is considered earlier in the SDLC, effectively “shifting left” and fostering a proactive security mindset among developers.

> **Aside**: Conway's Law posits that teams will architect software that mirrors lines of communication in the organization. The [Inverse Conway Manuever]({% post_url 2018-05-03-vsts-one-team-project-and-inverse-conway-maneuver %}) is a way to foster a cultural change by intentionally changing the architecture and tooling of your teams - so it's no surprise that using GHAS (which embeds security where the developers work) can be a powerful change agent to truly "shifting left" with regards to security. 

## Building a Strong Developer/Security Partnership: “You Own Your SDLC”

A collaborative partnership between DevOps and Cybersecurity is foundational to embedding security in the SDLC without disrupting workflows. By allowing teams to retain ownership of their SDLC while providing visibility into security issues and collaborative support, security teams foster trust and partnership rather than being seen as police. This approach empowers development teams to make informed security decisions while maintaining accountability for their codebase, helping eliminate the "blocker" stigma often associated with security.

## Distinguishing Between Risk Posture and DevOps Maturity

Separating risk posture from DevOps maturity provides clarity for both security leaders and development teams, allowing them to prioritize actions effectively. Risk posture assesses the actual security risks an organization faces, while DevOps maturity reflects the team’s ability to manage and remediate those risks efficiently. By focusing on DevOps maturity, security leaders can identify areas where process improvements could reduce risk over time, without conflating a low maturity score with an immediate risk issue. For teams, this distinction emphasizes continuous improvement rather than a reactive “fix everything now” mentality. Leaders benefit from this clarity as they gain insights into which teams may need additional support or resources to reach maturity, while developers are empowered to make incremental improvements, ultimately building a more resilient, secure SDLC. This approach aligns security improvements with business needs and development workflows, fostering sustainable growth in both security practices and development capabilities.

## Empowering Developers to Lead Security Initiatives

Empowering developers to take the lead in security efforts, such as through developer-led conferences and initiatives, encourages them to take ownership of security outcomes. When developers are provided with the opportunity to share successful practices and learn from their peers, they’re more likely to integrate security into their workflow naturally. By giving developers this responsibility, organizations not only enhance their security posture but also build a culture where security is recognized as a shared priority across all functions.

## Moving Fast As a Strategy for Better Risk Management

In a DevOps environment where code can be pushed daily, organizations can embrace a more agile approach to risk management. Rapid deployment cycles allow teams to fix issues quickly, potentially reducing the need to prioritize every security defect immediately. This flexibility enables a balanced approach to risk: teams that deploy frequently can accept some level of vulnerability knowing it can be promptly addressed, whereas teams with slower release cycles may need to prioritize fixes more urgently. Counterintuitively, improving velocity is a key enabler to reducing risk.

## Prioritizing Developer Experience (DevEx) in Cybersecurity

Improving the developer experience (DevEx) is essential for sustainable security integration. Historically, security teams focused primarily on identifying and addressing risk, often without fully understanding developers’ workflows. By prioritizing DevEx, cybersecurity efforts align more naturally with development processes, providing developers with tools, guidance, and autonomy that enhance productivity and foster a more collaborative security culture. Ultimately, the better an organization supports DevEx, the more effective it will be in achieving security outcomes without sacrificing speed or innovation.

# Conclusion

For organizations to truly "shift left", they need to move from a culture of "managing risk" to a culture of "improving DevOps maturity". Cybersecuritty teams need to understand the developer workflow and become enablers rather than blockers. By prioritizing collaboration, adjusting language, embedding security in the SDLC, and creating actionable metrics, organizations can build an AppSec program that enhances both security and productivity.

Happy securing!