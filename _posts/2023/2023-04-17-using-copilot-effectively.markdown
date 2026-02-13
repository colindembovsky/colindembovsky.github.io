---
layout: post
title: 'Using GitHub Copilot Effectively'
date: '2023-04-17 01:22:01'
image: /assets/images/2023/04/copilot-plane/copilot.jpg
description: >
  GitHub Copilot is an AI pair programmer that can dramatically increase developer productivity. However, it is still a tool - and developers must learn how to frame Copilot's capabilities in order to make the best use of it.
tags:
- development
---

1. TOC
{:toc}

> Image by [Rayyu Maldives](https://unsplash.com/@rayyu?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText") on [Unsplash](https://unsplash.com/photos/vZ5Tk3cc52o?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText")
  

GitHub Copilot is aptly named. While some have feared that generative models will replace developers, I do not believe we are there yet: Copilot is an assistant, not a replacement. However, developers will need to adjust their skills, both to stay effective as well as to stay marketable through the disruption that the AI age is bringing.

I have a friend who is a commercial airline pilot, and asked him about how autopilot works on commercial airplanes. I think the analogy of how autopilot works is useful in framing how developers should approach learning how to use GitHub Copilot.

## How Autopilot works on Commercial Flights

Most of us have flown in a commercial airplane. We all know that there are two human pilots in the cockpit, and we even know that they engage autopilot to fly the aircraft. However, even though we are all comfortable with the idea of planes flying themselves, we would be a little nervous if there were no humans in the cockpit before we take off!

Here is my understanding of how autopilot works during a commercial flights:

1. The pilot taxis the plane and takes off - the autopilot cannot take off automatically
1. Once the plane reaches around 15,000 ft altitude, the pilot engages the autopilot system. Some pilots will fly manually until they are at cruising altitude.
1. Once engaged, the autopilot is programmed to fly the plane along the current flight plan. The autopilot can navigate through bad weather and turbulence.
1. The pilots man the radios and watch for weather and wind conditions. At times, pilots will tell the autopilot to fly around weather, or change altitude to get better wind conditions.
1. The autopilot lands the plane.
1. The pilot takes over to taxi the plane.

> **Note**: Even if the above is not 100% correct, it's good enough to make an analogy for GitHub Copilot! Errors and omissions are my own.

## GitHub Copilot

Understanding how autopilot works, we can make a useful analogy when we consider GitHub Copilot:

1. Developers must "take off" since Copilot can't take off by itself (context)
1. Developers can use Copilot "mid-stream" but will need to make adjustments for "turbulence" (work in small chunks)
1. Developers must "man the radio" to monitor the code that they are writing with Copilot (good DevSecOps)) 
1. Copilot can "land the plane" but getting to the final destination is up to the developer (remember to solve the right problems)
1. Quality control is beyond the purview of Copilot

Let's dig into these a little deeper.

### Taking off - providing context

Just as autopilot can't take off automatically, in the same way a blank project or file isn't a good way to get going with Copilot. Even before that, developers need a "flight plan" - some idea of what they are going to be coding. Spending a little time to analyze requirements and think about how code is going to be written, tested, scanned, packaged and deployed will go a long way to better productivity and efficiency.

When using Copilot, you get the best results when supplying good context - think of this as the flight plan. _Context_ is the file that you're currently editing as well as other tabs open in the solution. If you have other files already, open a couple of them to assist Copilot. Open test files to help Copilot with tests and examples of how your methods are being called. 

Where none of this exists, take time to think about what you want the code to do and write the intent in comments at the top of the file. The more context you supply, the better your results will be.

I love doing the Advent of Code in December - and using Copilot while solving the puzzles has been great. However, I think one of the main advantages of using Copilot was that it subtly changed _how_ I develop: rather than simply diving into code, I take a few moments to think about how I can best prompt Copilot to give me what I want. This makes a big difference and I found myself spending more time thinking and less time thrashing code - which is a more fulfilling experience as well as a more productive way to code!

> Prompt engineering is a phrase that is being bandied about - I think there is something to this. Successful engineers will be those that can successfully guide AI to do the right thing.

### Cruising Altitude - working in small chunks

Once you have a little bit of code, you're at "cruising altitude". This is where Copilot feels like magic - there is enough context for it to generate the code that you were thinking of. Keep working in small chunks (like inside a method body or inside a loop). The narrowed context produces far better results.

Remember, Copilot is a _probability engine_ and there is some level of randomness inherent in how it works (this is true of all large language models). Broad, vague requests (low context) tend to produce results that show much more randomness (and less meaning and utility). Narrowing the context reduces the compounding effect of the randomness and is more likely to produce meaningful code.

### Man the Radio - fast feedback

While you're having fun coding with Copilot at your side, don't forget to "man the radio". Remember, code in an of itself isn't the goal - _solving business problems is_! Moving faster isn't an end - it's a means to an end.

Why do we want developers to be more productive and efficient? The value of going faster _is that we get feedback faster_. The faster we get feedback from our end-users, the faster we're able to adjust course. Scrum and Agile didn't succeed because of daily stand-ups and retros - Agile succeeded because it focused on flow and shortening the feedback loop. Copilot, by making developers more productive, is wasted unless you're shortening the feedback loop. Listen to the feedback from end-users, and adjust accordingly. This will give Copilot purpose and value beyond _just_ developer happiness.

### Land the Plane - Good DevSecOps

Landing the plane is crucial - after all, if your plane doesn't land, you can't get to your destination! Again, the landing of the plane is a means to an end - you have to get off the plane to reach your destination! 

Copilot will help you land, but you'll have to taxi in yourself. Copilot is designed to speed the "inner loop" of development - but you'll have to make sure you have an efficient "outer loop" too - peer code review, build automation, linting, unit and integration testing, scanning and automated deployment are critical if you're going to get the most out of Copilot.

> Having said that, some Copilot X features are bringing AI to the "outer loop" such as Copilot for PRs, which can suggest missing test cases for code changes in a PR.

### Autopilot is only for flying

Copilot allows developers to move faster - which means you need to match that speed when it comes to quality gates and deployment - otherwise you'll get an impedence mismatch, which if you know your electronics, is a Bad Thing. Copilot, by making developers faster, requires your quality gates and processes to be faster.

The autopilot on planes do not check the fuel levels or the ailerons or do any of the preflight checks itself - quality control is still up to the pilots and ground crews. Copilot is not meant to do everything for you - it's meant to augment your developers and make them faster. You must have good DevSecOps practices in place to maximize your usage of Copilot.

# Conclusion

GitHub Copilot is a powerful tool, but to get the most out of it developers should understand how to feed it context, work in small chunks, and ensure the rest of the DevSecOps pipeline is running smoothly. 

Happy co-piloting!