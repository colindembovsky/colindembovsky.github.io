---
layout: post
title: Project Plans, Agile, Pizza and Startups
date: '2014-04-14 17:49:00'
tags:
- alm
---

Last week I [posted about how to integrate TFS and Project Server “manually”](http://www.colinsalmcorner.com/2014/04/integrating-tfs-and-project-server-two.html). In the post I did put in a bit of philosophy about why I think project plans can be a Bad Thing. Prasanna Adavi posted a thoughtful comment on my philosophy, and I wanted to reply just as thoughtfully, so I decided to add the reply as a new post rather than just reply to the comments inline.

Here is Prasanna’s comment:

“I disagree with you on your notion of how "project plans" work. It is wrong to assume that just because a project manager plans and baselines a project, they will never veer from it. To the contrary, It is an attempt to make the customer think before hand as to what they perceive as the value, and then think twice before they "change" the requirements. A good project manager is never afraid to reset the baseline if it is necessary to do so.

Moreover, who said that a "waterfall" has to be one huge waterfall? Depending upon the nature of the project (especially Software Projects), they can be scheduled as multiple mini-waterfalls, yielding quick, unique features. Actually, the way I see Agile is multiple mini-waterfalls. But within each waterfall, there needs to be a commitment. Leaving it open ended and reworking things again and again, because customer does not know what he wants, is not really 'embracing change'.

Think about this: When you order a pizza, do you 'commit' to certain toppings, or do you say lets start making the thing, and we will add toppings as we see the value?

And finally, If your entire team or company is working on one software project/feature set, then it is great to think in terms of "Agile". But when you are on a 'agile' project that never ends because a customer constantly keeps seeing 'value' in the latest change, that is not a project anymore, and it becomes quite difficult to commit to anything else.”

## Response

Of course when I speak about Project Managers and Project Plans, I generalize. Some Project Managers are obviously better than others. In my experience, however, Project Managers that manage software projects still use the wrong paradigm when they produce a plan – and the fundamental paradigm of a Project Plan is that _we know what we’re building_ and we know _how long each task is going to take_. If either of these suppositions is incorrect, then the Project Plan’s use diminishes dramatically. Of course in Software Development, it’s nearly always the case that at the beginning of a project (when we’re supposed to be creating the plan) that requirements cannot be known fully – and we can guess how long each task will take, but it’s just that – a guess.

I like to show a diagram called the “Cone of Uncertainty” when I talk about Project Planning:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-jVMD_3vJvH0/U0uhIocjoUI/AAAAAAAABR8/KPbzX0nT3qA/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-Ano-qO91NKg/U0uhHfRlt2I/AAAAAAAABR4/552TpI5ifeQ/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

The Cone shows how the larger the “thing” we’re estimating is (and the further in the future the end date will be), the more uncertainty there is in that estimation. So planning something small right now has little variance – we might say that changing a table in a database will take 2 hours (with about half an hour variance). But if we plan something big (like a Project) and estimate 3 months, then the variance is likely to be +- 1.5 months. We just don’t know enough now to be more accurate.

This is why Agile is about working in small sprints (or iterations) between 1 and 4 weeks in length. That time period is a good fit for the kind of estimations that generally apply to detailed tasks on a software project. Planning further ahead (in detail) is simply wasted effort. Of course, if you don’t have any plan further out than your current sprint, that’s a bad thing too – you need to be working towards some larger goal or initiative. So Prasanna is theoretically correct that Agile is a series of mini-waterfalls – in the sense that the larger Project is divided up into smaller iterations. However, to put a lot of effort into Planning all the iterations in detail is simply a waste of time – things change so rapidly that by the time you’re half-way into iteration 1, the plan for iteration 3 is obsolete.

So being Agile doesn’t imply “not planning”. It just says, to use the 80/20 principle, put 80% of your planning effort into the 20% that matters most – in this case, that’s the current sprint. So plan in detail your next sprint, but wait till you’re nearing the end of sprint 1 before you plan sprint 2 and so on.

Let’s imagine that you’re on a team working in 2 week sprints. Now if you’re only planning in detail 2 weeks at a time, do you really need a Project Plan? Isn’t a list of requirements and their tasks (a backlog) enough?

The point I was making on my previous post was that putting a lot of effort into a large and detailed project plan, and then artificially sticking to that plan, is a waste of effort. And if you’re going to change your baseline often, then why baseline in the first place? Rather plan in detail in the short term (the next sprint) and let changes that come from that spring influence the detailed plan for the next.

Another point is that when you adjust your sprint based on feedback, then you’re “responding to change”. Normally a baseline changes because things are going to take longer than you expected – so the change isn’t based on change (or value), it’s based on poor estimates.

## Commitment

Of course just because you’re an Agile team that can respond to change rapidly, that doesn’t mean that you’re at the mercy of the customer’s whims. One of the principles of Agile is to _lock down the sprint_. So if the customer does decide to change things, they’ll have to wait till at least the next sprint. And of course, since the Product Owner (who represents the customer) is the one that needs to prioritize work, they’ll have to bump something off the backlog if a new change or requirement is added in. In fact, I’d go so far as to say that if your team is doing Agile correctly, you’ll get fewer change requests from your customers and they’ll soon learn to be a lot clearer about what they want and be more sure that they actually want it. Having a detailed Project Plan isn’t necessarily going to make your customers think more about what they want – even if baselines move. But giving them responsibility, having them see that every change shifts the goal-posts, will mean they will feel the change much more “personally”. Also if you deliver something (even if it’s small) at the enf of every sprint, customers get a feeling of momentum. You may even find that the seemingly small piece of functionality that you deliver is enough, and the customer doesn’t really need the rest of The Thing that you would have been working on for the next 2 months. _Rapid feedback on value delivered is far better than explaining why a baseline has to move_.

## Pizza or Degree?

Finally, the Pizza analogy. Again, if you go back to the Cone of Uncertainty, there is very little “risk” in getting toppings wrong on a pizza – so of course you commit to the entire pizza when you order it. I think the analogy doesn’t really work. I think a better analogy is a startup company.

Let’s imagine 2 startups – one called ProjectX and one called AgileX. ProjectX spends 4 weeks getting a detailed plan in place for their product, which is going to take 3 months to build. AgileX in the meantime spends 4 days on a general direction, and a day in detailed planning for their project. They release their product’s V1 after another 2 weeks. The product isn’t fully featured, but what there is of the product works. They get some feedback which changes some of what they know about the product, and adjust accordingly. If they continue to add small features and get feedback in 2 week cycles, they would have released features 15 times before ProjectX’s product comes out the door. They would have had 14 opportunities for feedback from their Customers before ProjectX even had 1 round of feedback on working software. Even if ProjectX followed Prasanna’s “mini-waterfalls”, there’d be less movement. A mini-waterfall would require too much planning for 2 weeks, so ProjectX decides on 3 1-month mini-waterfalls. That still means that they only release 3 times in the same period, and only get feedback on 2 occasions. Still way less than AgileX. Is the more detailed longer-termed planning helping or hindering?

What do you, dear reader, think of Project Planning?

Happy planning!

