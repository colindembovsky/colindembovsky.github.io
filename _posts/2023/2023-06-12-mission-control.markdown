---
layout: post
title: 'Mission Control - and what it means for DevSecOps'
date: '2023-06-12 01:22:01'
image: /assets/images/2023/06/missioncontrol/army.jpg
description: >
  Culture, culture, culture - it eats DevSecOps for breakfast! But what sort of culture should organizations build to succeed at DevSecOps? In this post I take a look at Mission Control and what it means for DevSecOps culture.
tags:
- devops
---

1. TOC
{:toc}

> Photo by <a href="https://unsplash.com/@fandrejevic?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Filip Andrejevic</a> on <a href="https://unsplash.com/s/photos/army?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

Today's markets move fast. Organizations that don't keep pace are being left behind. DevSecOps is fairly easy to grasp conceptually, but is not easily implemented. Most organizations that struggle to implement DevSecOps effectively are hampered not by tooling, but by old ways of thinking.

DevSecOps requires a cultural shift - as well as a platform to support this shift. A reminder of Donovan Brown's definition of DevOps is warranted:

> DevOps is the union of people, process and products to enable continuous delivery of value to our end users.

We used to say it this way when I was a DevOps consultant:

> You can't but DevOps in a box.

There is no "silver bullet" or product that will "make you DevOps". Finding the right tools and platforms is important, but culture is more so. Many teams talk about "technical debt" but I don't hear a lot of teams talk about _process debt_.

## Roots of Process Debt

There are probably many roots of process debt, but I think that many of them come from the Waterfall mindset. In Waterfall, the idea was to work out all the possible scenarios and outcomes up-front so that we could minimize risk. Ironically, this extreme "analysis paralysis" almost always led to _building the wrong things_ which was the exact thing it was trying to prevent!

A second factor was the desire to find economies of scale. For example, it was common to have database administrators (DBAs) and security professionals that took care of all the database and security work respectively. "Developers don't know how to optimize database work, so we'll centralize that work to let the developers code faster." Again, the irony is that DBAs became a bottleneck. The same is true of security teams - the desire to "offload" security from App Teams ends up slowing teams down!

As I was thinking about process debt, I came across a philosophy from the US Department of the Army called Mission Control that seemed to offer some insights into how to build a good DevSecOps culture.

## Army Mission Control

> Mission Control is the Army's approach to command and control that empowers subordinate decision-making and decentralized execution appropriate to the situation.

In war, events are too chaotic and communication too fragmentary to rely on centralized control. Commanders need to rely on the _innovation and decisive action of subordinates to meet intent in a complex operating environment_. Sounds like this applies to DevSecOps, doesn't it?

The seven principles of Mission Command are:

- **Competence** - developed continually through training and self-development of soldiers
- **Mutual trust** - shared confidence between soldiers and commanders that they can be relied upon and are competent to perform assigned tasks
- **Shared understanding** - creating common language and culture and clear visions and values
- **Commander's intent** - commanders must clearly communicate intent to everyone, articulating purpose and desired end state
- **Mission orders** - describing the situation, commander's intent, desired results and required tasks, _without specifying how tasks are to be accomplished_
- **Disciplined initiative** - whether the benefits of a localized decision outweigh the risk of desynchronizing the overall operation, and whether the action further's the commander's intent
- **Risk acceptance** - commanders must assess risk to mission while mitigating risks with control measures, trusting that their intent has been relayed and subordinate decisions will be made based on that intent

## Applying Mission Control to DevSecOps

We can apply these principles to our thinking about culture for DevSecOps.

### Competence

Investing in people and their skills is a critical part of a successful DevSecOps culture. Developers need to be empowered to learn about new technologies, stacks and trends. Similarly, cross-functional teams need to have training available for the breadth of their responsibilities. These responsibilities go beyond just coding and include testing, automating, monitoring, security, hyper-scale, infrastructure as code, cloud operations, live-site culture and more. By investing in training and opportunities for learning, companies build competence.

### Mutual trust

Trust is critical - but it must be _mutual_. App Dev teams must trust that their commanders (executives) are investing in them, and executives must trust their teams to do the right thing. This trust is earned and built over time, and can only be built on a culture that values innovation and won't punish people for initiative.

### Shared understanding

Executives must clearly communicate the vision and values of the organization so that it is well understood by everyone. Organizations should also spend time thinking about a common language as well as communication lines and types (see the three key [Interaction Modes](https://teamtopologies.com/key-concepts) from Team Topologies). Organizations that are clear about how they communicate can take advantage of the homomorphic force of Conway's Law to ensure that their architectures and culture are aligned, rather than opposing.

### Commander's intent

Beyond the values and vision, executives must clearly communicate _purpose_ and _desired end state_. Clearly articulating what success looks like and what the key objectives are at the executive level keeps everyone aligned.

In my [previous post]({% post_url 2023-06-07-scaling-dev-sec-ops %}) I spoke about the balance between _Team autonomy_ and _Enterprise alignment_. When executives are crystal clear on the purpose of an organization as well as desired end state, this gives teams strong enterprise alignment. Strong enterprise alignment at the _strategic level_ promotes a culture where Teams feel empowered to innovate within the boundaries that the organization really cares about.

### Mission orders

This is where most organizations get it wrong - mission orders are about distilling the commander's intent, in language built from Shared understanding, and specifying _what_ needs to be done, not _how_ it should be done.

This requires the vision, purpose and values of the organization to be clearly understood. It requires good shared understanding, but it is also built on mutual trust. Will leaders trust that their teams have the competency to do what needs to be done? Do developers trust that the leaders are investing in them?

This also ties back well to Enterprise alignment - which emphasizes a core set of non-negotiables (the values) and lets teams innovate within these parameters to meet the Commander's intent (Team autonomy).

### Disciplined initiative

If organizations have clear Mission orders, know the Commander's intent, and what they are tasked to achieve, _then they can innovate to fulfil the goals_. Rather than specifying _how_ they should do things, which signals a lack of trust, executives show trust by letting teams apply initiative. This benefits the teams (they gain mastery and autonomy) and the company, since the company is now building a culture of innovation. This then feeds to better trust, which leads to more autonomy - and the virtuous cycle continues.

Team autonomy is what is being expressed here - within the boundaries of clear, concise Enterprise alignment.

### Risk acceptance

This is a tough one for most organizations. However, if the other principles are in place, then this becomes the natural progression. Organizations that have a low-trust environments tend to be highly risk-averse.

This is not to say that risks should not be evaluated, weighed and mitigates when appropriate. However, teams that default to zero risk also stifle innovation and experimentation. When organizations build competent teams in a high-trust environment, are clear about their purpose and vision, then they can accept the risk of _letting teams fail_. If teams are never allowed to fail, they will never innovate. Once again, settling on a small core of non-negotiables (Enterprise alignment) and then giving teams room (Team autonomy) to innovate, experiment and (at times) fail, shows trust.

## Conclusion

The principles of the Army's Mission Control philosophy apply well to the culture of DevSecOps. Organizations that want to succeed need to develop a culture that builds mutual trust and empowers innovation, rather than stifling it.

Happy missioning!