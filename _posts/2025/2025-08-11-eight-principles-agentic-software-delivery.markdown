---
layout: post
title: Eight Principles for Agentic Software Delivery (ASD)
date: '2025-08-12 00:30:00'
image: /assets/images/2025/08/agents-humans.png
description: >
  Learn eight practical principles for evolving your software delivery with AI-driven capabilities while maintaining human oversight and delivering real business value.
tags:
- ai
- devops
---

1. TOC
{:toc}

In this post, I'll show you eight principles for implementing Agentic Software Delivery (ASD) and why they matter for accelerating value delivery while maintaining quality and security. You'll learn how to blend human expertise with AI capabilities throughout your SDLC, evolving a delivery system that's faster, smarter, and more focused on business outcomes.

## The Evolution from Continuous Delivery to ASD

Right as DevOps was becoming an industry standard, Continuous Delivery gave us some good [foundational principles](https://devopsnet.com/2011/08/04/continuous-delivery/) to put into practice: automate everything, maintain quality, and deliver frequently to name three. In one of my previous posts, I define [Agentic Software Delivery]({% post_url 2025-05-01-agentic-software-delivery %}) (ASD) but in this post I want to start making it more practical by providing a set of principles. For each principal I want to assess its impact on the three pillars of ASD and show some practical GitHub implementation tips.

## Principle 1: Outcome-Focused Delivery

Prioritize business outcomes over mere output. Every feature should tie back to customer value, and "done" means value is delivered in production, not just code completed. This may seem obvious, but there is a huge focus on which models write better code or what percentage of your codebase is written by AI - all of which are meaningless numbers if you are not achieving business outcomes. The goal isn't more AI - it's better results, improved productivity and happier developers.

### GitHub Application
- Tie your work item tracking to Business Objectives so that the business outcome is clear throughout the SDLC
- Configure GitHub Advanced Security to track vulnerability fixes as business outcomes - security is business value
- Don't overindex on a single measure - think system wide (read the GitHub [Engineering System Success Playbook](https://resources.github.com/engineering-system-success-playbook/))
- On an architectural level, you should validate business value during rollout. One way could be to use feature flags and partial deployments to validate business metrics before full rollout.

### Impact Classification
- **Human Expertise**: HIGH - Humans define business value and success metrics
- **Autonomous Agents**: LOW - Agents execute but don't determine business priorities  
- **Intelligent Context**: MEDIUM - providing business value goals in agent-readable (and human readable) formats will improve agentic results

## Principle 2: Human-AI Collaboration by Design

Integrate human expertise with AI at every stage. Design processes where routine tasks are handled by AI while complex decisions remain human-guided. The goal is not to replace humans, but to replace mundane and low-level toil tasks with AI so that humans can work at more abstract layers and do higher-order work.

### GitHub Application
- Enable GitHub Copilot for AI pair programming across your organization. This requires more than just assigning a license - you need to create enablement programs (and ideally, teams) that will continually train and enable teams. AI is moving fast, and just like we need Continuous Delivery, AI development requires Continuous enablement.
- Use Copilot Code Review to assist with the increase in review burden
- Use Copilot to generate READMEs, architectural documentation, style guides, coding patterns and other "tribal knowledge". Source control these documents alongside code so that Copilot has access in the IDE or through Coding Agent.
- Use Copilot to generate [custom instructions](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions?versionId=free-pro-team%40latest&productId=copilot&restPage=tutorials%2Ccopilot-chat-cookbook) and custom [Chat Modes](https://code.visualstudio.com/docs/copilot/chat/chat-modes) that tailor and personalize Copilot
- Invest in good automated quality gates, applied at scale using [Rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets). Make the gates thorough so that you have high trust in code that passes the gauntlet. Apply these gates irrespective of the _source_ of changes (human, AI or a mix).
- Create [Copilot Spaces](https://github.blog/changelog/2025-05-29-introducing-copilot-spaces-a-new-way-to-work-with-code-and-context/) that encapsulate softer skills, general coding guidelines and standards, and other "tribal knowledge" so that you can build a library that is searchable and personalized

### Impact Classification
- **Human Expertise**: HIGH - Humans provide context and custom guidelines, make decisions, and validate AI output
- **Autonomous Agents**: HIGH - Agents handle repetitive coding and testing tasks; automated gates do much of the validation heavy-lifting
- **Intelligent Context**: HIGH - AI is personalized to your codebase and team patterns

## Principle 3: Intelligent Automation Across the SDLC

Automate everything feasible and use AI to extend automation into complex, context-driven tasks. Go beyond simple CI/CD to intelligent pipeline optimization. Teams that use build automation always outperform teams that build manually: the same will be said of teams that leverage AI in their pipelines to keep pipelines running continuously - they will outperform teams that rely on "traditional", static pipelines.

### GitHub Application
- Implement GitHub Actions workflows that [self-heal]({% post_url 2025-08-08-self-healing-devops-with-copilot-and-actions %}) based when they fail
- Use Dependabot for automated dependency [updates](https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/about-dependabot-security-updates) with intelligent grouping
- In high-activity repositories, deploy GitHub's [merge queue](https://github.blog/engineering/engineering-principles/how-github-uses-merge-queue-to-ship-hundreds-of-changes-every-day/) to prevent failing PRs from blocking the entire pipeline
- Use [GitHub Hosted Runners](https://github.blog/enterprise-software/ci-cd/when-to-choose-github-hosted-runners-or-self-hosted-runners-with-github-actions/) so that you can concentrate on software delivery rather than trying to scale and manage build farms

### Impact Classification
- **Human Expertise**: LOW - Humans set policies but don't manage execution
- **Autonomous Agents**: HIGH - Agents handle most automation tasks independently
- **Intelligent Context**: HIGH - AI is leveraged to self-heal tests, pipelines and other automated processes

## Principle 4: Single Source of Truth and Platform Integration

Keep all code, configurations, and documents in a single system for consistency and traceability. This enables both humans and AI to work from the same context. There may be exceptions that are role-dependent: designers typically work in tools like Figma rather than in source control systems like GitHub. For external systems, leverage Model Context Protocol (MCP) to provide context or extend model capabilities (tools). However, the backbone of your development system should be a single, AI-powered Source Control Management (SCM) system.

### GitHub Application
- Choose a single SCM tool - make sure this platform is capable of running intelligent pipelines and integrating to other systems when necessary. Make sure this is a tool that your developers will enjoy working with!
- Store infrastructure as code alongside application code in GitHub repos
- Implement GitHub's CODEOWNERS for clear ownership of code
- Reduce the number of tools in your ecosystem, and use platform-native tools where possible. This not only reduces cognitive load and the cost of integration, but the fewer external systems you need, the faster you can get context from those external systems to agents when needed.

### Impact Classification
- **Human Expertise**: MEDIUM - Humans establish structure and governance
- **Autonomous Agents**: HIGH - Agents need unified access to operate effectively
- **Intelligent Context**: HIGH - Centralized data and reduced integration footprint enables better AI understanding

> Note: A fragmented toolchain limits AI effectiveness. Consolidation isn't just about efficiency; it's about how quickly you can enable AI agents to understand your entire system.

## Principle 5: Built-In Quality and Security

Embed quality and security from the start. AI tools should help generate tests, detect vulnerabilities, and enforce standards continuously.

### GitHub Application
- Enable [GitHub Advanced Security](https://docs.github.com/en/enterprise-cloud@latest/get-started/learning-about-github/about-github-advanced-security) for automatic vulnerability scanning and remediation at scale using [Campaigns](https://docs.github.com/en/code-security/securing-your-organization/fixing-security-alerts-at-scale/about-security-campaigns?versionId=free-pro-team%40latest&productId=copilot&restPage=tutorials%2Ccopilot-chat-cookbook) and [Autofix](https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/responsible-use-autofix-code-scanning)
- Configure [secret scanning](https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning) with custom patterns for your organization
- Implement Actions workflows that block deployments on security issues
- Leverage GitHub Copilot to create unit and integration tests

### Impact Classification
- **Human Expertise**: HIGH - Humans define quality standards and security policies
- **Autonomous Agents**: HIGH - Platform continuously scans, while Copilot continuously improves test coverage
- **Intelligent Context**: MEDIUM - AI uses common test patterns and systems effectively

## Principle 6: Shared Responsibility and AI Governance

Everyone shares responsibility for success, including AI systems. Establish clear governance for what AI can do autonomously versus what requires human oversight.

### GitHub Application
- Set up CODEOWNERS to require human review for critical paths
- Focus on creating reusable documentation that can guide agents effectively so that they can work more independently

### Impact Classification
- **Human Expertise**: HIGH - Humans maintain accountability and set boundaries
- **Autonomous Agents**: MEDIUM - Agents operate with clear instructions
- **Intelligent Context**: MEDIUM - Common practices and guidance is shared for humans and agents

## Principle 7: Parallel Experimentation

Leverage the scale of autonomous agents to experiment widely. Without AI, creating several solutions to a problem means utilizing several teams, or the same team to solve the same problem several times. This is cost-prohibitive. But with agents, you can instruct several agents to work on different flavors of a solution in parallel and pick the best one, since cost is not longer a limiting factor.

### GitHub Application
- Automated pipelines is critical for parallel experimentation, since the object is to have each solution be created independently and autonomously
- Leverage Coding Agent and instead of requesting 1 solution, create 3 or 5 Issues each with a different "flavor" of solution (using Agent Mode to brainstorm) and assign Coding Agent to each Issue. Pick the best result and discard the others.

### Impact Classification
- **Human Expertise**: MEDIUM - Humans interpret results and make decisions
- **Autonomous Agents**: HIGH - Agents brainstorm, plan and implement multiple solutions in parallel
- **Intelligent Context**: MEDIUM - AI leverages business outcomes, custom instructions and other artifacts to produce solutions

## Principle 8: Continuous Learning and Adaptation

Commit to evolving both your process and your AI tools and integration. Feed learnings back into the system so it gets smarter over time.

### GitHub Application
- Use GitHub Discussions to capture retrospectives and learnings. Distill key insights into improved documentation, instructions and Spaces
- Track metrics in GitHub Insights to identify enablement opportunities

### Impact Classification
- **Human Expertise**: HIGH - Humans drive improvement initiatives and learning
- **Autonomous Agents**: MEDIUM - Agents improve as guidelines are improved
- **Intelligent Context**: HIGH - AI models improve through continuous improvement and assessment

> Note: Your delivery system should be treated as a product that you continuously refine. Regular retrospectives should include evaluating AI performance alongside human processes.

## Putting It All Together

These eight principles work together to create a delivery system that is more than the sum of its parts. When you combine outcome focus with AI collaboration, a unified platform with built-in quality, and experimentation at scale, you get an SDLC that delivers value consistently and adapts to changing needs. I believe that GitHub is uniquely positioned to empower this transformation.

## Getting Started with ASD

Start small. Pick one or two principles that address your biggest pain points:
- If you're drowning in repetitive tasks, focus on Intelligent Automation
- If quality issues slip through, prioritize built in Quality and Security
- If you're not sure you're building the right thing, start with Outcome-Focused Delivery

Remember, ASD isn't about replacing your existing practices overnight. It's about gradually evolving them to leverage AI capabilities and augmenting the human expertise that makes your team unique.

## Conclusion

Agentic Software Delivery represents the next evolution in how we build and deliver software. By following these eight principles and leveraging platforms like GitHub that support both human and AI collaboration, you can create a delivery system that's faster, smarter, and more focused on what really matters: delivering value to your business and users.

Happy delivering!