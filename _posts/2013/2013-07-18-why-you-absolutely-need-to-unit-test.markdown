---
layout: post
title: Why You Absolutely Need to Unit Test
date: '2013-07-18 20:56:00'
tags:
- build
- testing
- development
---

I’ve written about why builds are absolutely essential in modern application development ([Part 1](http://www.colinsalmcorner.com/2013/06/automated-buildswhy-theyre-absolutely.html)) and what why Team Build is a great build engine ([Part 2](http://www.colinsalmcorner.com/2013/06/automated-buildswhy-theyre-absolutely_18.html)). However, if you don’t include unit tests in your builds, it’s like brushing your teeth without toothpaste – there’s a lot of movement, but it’s not the most effective way to do things. In this post, I want to put forward a few thoughts about why you absolutely need to be unit testing.

Here’s the highlight list:

1. Coding with unit testing in mind forces you to think about the design and architecture of your code – making it better code
2. Unit testing provides immediate feedback – every change in code is tested (at least partly) as you’re coding
3. A small investment now leads to a massive saving later – bugs in production cost way more than bugs in development
4. Unit testing provides metrics for the quality of your code
5. Unit testing builds inherent quality into your releases

Let’s consider each of these statements.

## Think-Driven Development

You’ve probably heard of terms like “test-driven development (TDD)” or “behaviour-driven development (BDD)”. I’d love to coin another term – _think-driven development_. I come from a development background – and since I love coding (as most developers do), I suffer from “never-mind-the-details-I-just-want-to-start-coding” syndrome. Most developers I meet do. Hopefully you never lose this passion – but there’s a lot to be said for taking a breath and thinking first.

If you just jump into the code, and you don’t have tests, you’ll probably hack something out that will work initially for a couple of scenarios (usually with the attendant caveat, “it works on my machine!?!”). However, this isn’t sustainable since it leads to what one of my [university lecturer](http://www.ru.ac.za/computerscience/people/staff/alfredoterzoli/)’s used to call “spaghetti code” (which was especially funny since he’s Italian). You write some code and get it deployed. Something breaks. You band-aid it and re-deploy. Now you’ve broken something else. So you add some sticky-tape (metaphorically, of course). Eventually you’re down to chewing gum and tin-foil, and things are going from bad to worse.

If you start out writing tests (or at least start writing your code with testing in mind), you’re more likely to come up with better code. If you’ve every tried to write unit tests for legacy code, you’ll understand what I mean. If you don’t code with tests in mind, not surprisingly, your code ends up untestable (or at least really hard to unit test). Writing testable code forces you to think of good decomposition, good interfaces, good separation of concerns, inversion of control and dependency injection and a whole slew of other principles we all learn about but somehow forget to use in our daily grind.

So start with your tests. This forces you to use all of the good stuff (they’re not called _best practices_ for nothing). The little bit of thinking required up-front is going to save you a whole lot of pain down the line (not to mention decrease the amount of chewing gum you find in your source repository).

## Immediate Feedback

Let me ask you a question – how long is the feedback loop between the time you write some code and when you get feedback about the validity of that code? Or phrased another way, think about the last bug you fixed. What’s the length of time between the writing of the code and the report of the bug? One day? One week? One year?

The fact is that the sooner after writing code that you find a bug, the cheaper it is to fix. Let’s consider two ends of the spectrum: coding-time and in production.

When a bug is found in production, it can take a while to find it. Then it’s reported to a service desk. They investigate. They then escalate to 2nd line support. They investigate. They then escalate to the developers, who have to investigate and repro. In other words, a long time.

What about when you’re coding? You write some code and run your unit tests. The test fails. You investigate and fix. In other words, a really short time.

The quicker you get feedback (and the more often you get feedback) the more efficient and effective you’re going to be. On a large scale, this is broadly the reason for Agile methodologies – the rapid iteration increases the frequency of the feedback loop. It works at a project management level, and it works at a coding level too.

Here’s how that looks on a graph:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-Vkm4wDuUYPM/UefXzn1D3hI/AAAAAAAAA-Q/MV_eRfIz-b8/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-yQl5GxEmUC4/UefXy1Vb9yI/AAAAAAAAA-I/9B7Kov4SEqU/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

The earlier you find your bugs, the less time you’ll spend finding them and the cheaper it is to fix them. If you don’t have unit tests, you’re already moving your first feedback opportunity to the 3rd zone (manual testing). If you don’t do manual testing, you’re pushing it further out again into UAT. If you don’t do that – then the 1st feedback you’re going to get is from production – which is the most time-intensive and the most costly.

Getting immediate feedback while you’re coding is great when you’re doing “greenfield” projects (projects that have never been deployed before. It really starts to shine when you come back after 6 (or 12 or 18) months to add features – you still have your suite of tests to ensure that you’re not breaking anything with your new code.

## Spend a little now – save a lot later on

Almost without fail the teams that don’t unit test claim that “they don’t have time to unit test”. I argue that in the vast majority of such cases, it’s exactly because you don’t invest in unit tests that you don’t have time to code properly. Every time you deploy untested code, you increase your technical debt. The sad fact is that technical debt tends to grow exponentially.

Look again at the graph above. Where do you think you’ll spend more time finding and fixing bugs? Obviously the further “to the right” you are, the more time you need to fix a bug. So invest a little “now” while you’re coding, so that you don’t have to spend a lot of time later on dealing with production issues!

## Quality Metrics

How do you measure the quality of your code? Bugs in Production per time-period? Mean Time to Resolve (MTTR)? Most of these measurements are helpful to some degree, but because they’re “production time” statistics, the feedback is too late in the cycle.

The teams I work with that do have metrics invariably have unit test metrics – pass / fail rates and code coverage statistics. The teams that don’t have metrics almost always don’t have unit tests. Unit tests provide you with an objective measure of the quality of your code. How can you improve if you don’t know where you currently are?

What coverage should you be aiming for? This is debatable, but I always recommend that you rather concentrate on your trends – make sure each deployment is better than the last. That way the absolute number doesn’t really matter. But the only way you can do any sort of trend analysis is if you’re actually collecting the metrics. That’s why I love unit tests (with code coverage) in TFS Team Builds, since the metrics get collated into the warehouse and it’s really easy to do trend reports. Showing business a steadily improving graph of quality is one of the best things you’ll ever do as a developer!

## Inherent Quality

All of these practices lead to _inherent_ quality – a quality that’s built into what you’re doing day-to-day, rather than something you tack on at the end of an iteration (or quickly before release). If you build inherent quality into your processes and practices, you can sleep soundly the night of your big deployment. If you don’t, you’ll have to keep the phone close-by for all those inevitable calls of, “Help, the system is broken…”

## Legacy Code

In my previous company we had large amounts of “legacy” code – code that was in production, that we constantly supported, that had no tests. When I started pushing for unit tests and coverage, we inevitably got to the awkward moment where someone blurted, “There’s just too much code to start testing now!”. I argued that 1 test was better than 0 tests. And 2 tests are better than 1 test, and so on.

We then made it a policy that whatever code you worked on (be that new features or bug-fixes) needed to be unit tested. This started us on the path to building up our comprehensive test suite. We also put a policy in place that your code coverage had to be higher for this deployment than the previous deployment in order for your deployment to be approved. At first our coverage was 1.5 or 2%. After only 6 months, we were somewhere in the 40%’s for coverage. And the number of production issues we were working on was decreased dramatically.

## Conclusion

Just like you can’t afford not to do automated builds, you really can’t afford not to do unit testing. The investment now will provide massive benefits all along your application lifecycle – not only for you and your team, but for your business and stakeholders too.

For more reading, Martin Hinshelwood wrote a [great post](http://nakedalm.com/you-are-doing-it-wrong-if-you-are-not-using-test-first/) about test-first that I highly recommend.

Happy testing!

