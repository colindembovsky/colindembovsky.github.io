---
layout: post
title: Monitoring Web Applications – Continuous IntelliTrace
date: '2013-09-27 20:01:00'
tags:
- alm
---

If you have [Visual Studio Ultimate](http://www.microsoft.com/visualstudio/eng/2013-downloads) and are not using [IntelliTrace in production](http://msdn.microsoft.com/en-us/library/vstudio/hh398365.aspx), you should be drawn and quartered. This is arguably the best feature of Visual Studio Ultimate, and in my opinion this feature alone justifies the pricing (never mind Web Performance and Load testing, Code Maps, Code Lens, UML diagrams and Layer diagrams).

The standalone IntelliTrace collector is amazing, and will run anywhere. It’s especially useful for diagnosing problems in Web Applications running in IIS. (For a recap on how to use this tool, see my series starting [here](http://www.colinsalmcorner.com/2013/06/intellitrace-tips-and-tricks.html)).

Often when I talk about collecting IntelliTrace logs, people invariably ask, “Why can’t I leave the collector running all the time?”. This depends on how resilient you are to the performance impact of the collector, as well as what “mode” you’re using. If you using the verbose “Call and Events” mode, you’re going to see a performance knock. If you use the “Events Only” mode, you may see less impact (of course the logs won’t be as rich) but even this can be too much degradation.

About a week ago, Microsoft released the [Microsoft Monitoring Agent](http://blogs.msdn.com/b/visualstudioalm/archive/2013/09/20/introducing-microsoft-monitoring-agent.aspx). This agent can be run “standalone” or be connected to System Center Operations Manager (SCOM) 2012 R2. I’m going to show you how you can run this agent in standalone mode in this post.

## Why use the Monitoring Agent?

Ah, we get to the crux of the matter – why use this agent instead of the IntelliTrace standalone collector? The answer is two-fold:

1. You get performance monitoring “for free” when you use the Monitoring Agent
2. You can leave the agent on – permanently
3. You can target a specific web application (instead of a whole application pool)

Unfortunately (for some) the agent is something you install – not like the IntelliTrace standalone collector that is just xcopy-able. If installing agents in your production environments is not a challenge, then you should be switching to the Monitoring Agent.

## Running Continuously

There’s a caveat to running the monitor continuously (isn’t there always?). You really only want to do this in one of the three monitoring modes available – “monitor” mode. (The other two are “trace” and “custom”).

The IntelliTrace standalone collector comes with two xml configuration files out-the-box: events only (default) and events and call information (trace). “Trace” mode will give you the same as the “events and call information” mode of IntelliTrace standalone collector – it’s verbose, but it’ll knock your performance. You’ll have to be selective about when you run this mode. The “custom” mode let’s you run the collector using a custom IntelliTrace xml file, so you can tailor the logging just so.

The Monitoring Agent’s “monitor” mode is like the events only (default) setting of the IntelliTrace collector, but even more lightweight. Instead of collecting all events, it only collects exceptions that bubble up to the global exception handler and “slow” events (events that take the server longer than 5 seconds to respond). Also, it’ll only record 60 triggers of each event daily. And since you can specify which web site to monitor, you don’t have to impact your whole application pool.

The upshot of this is that you can turn monitoring on – and leave it on. Then you can “checkpoint” the log at any time – either when there’s a problem or proactively whenever you want to.

## Running the Monitoring Agent

I downloaded and installed the Monitoring Agent from the [download site](http://www.microsoft.com/en-us/download/details.aspx?id=40316). I ran the installer, and configured it to run standalone (i.e. skipped hooking it up to SCOM). Thereafter I opened a PowerShell command prompt, but the module isn’t imported for some reason. No worries – just run this command:

Import-Module installPath\Agent\Microsoft.EnterpriseManagement.Modules.PowerShell.dll

From there, you can follow Larry Guger’s post to see how to [run the agent](http://blogs.msdn.com/b/visualstudioalm/archive/2013/09/20/introducing-microsoft-monitoring-agent.aspx), checkpoint the log and review the results (especially [performance events](http://blogs.msdn.com/b/visualstudioalm/archive/2013/09/20/performance-details-in-intellitrace.aspx)).

Happy monitoring!

