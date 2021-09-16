---
layout: post
title: 'Imaginet Timesheet: Time Tracking for TFS and Visual Studio Online'
date: '2014-05-30 02:33:06'
tags:
- news
---

We’ve been working on a rewrite of our Timetracking tool (formerly Notion Timesheet) and it’s going live today – [Imaginet Timesheet](http://www.imaginet.com/imaginettimesheet)! Timesheet lets you log time against TFS work items using a web interface. The web site can be installed on any IIS server (if you want to host it on-premises) or even onto Windows Azure Web Sites (WAWS) if you have a public-facing TFS or are using Visual Studio Online. Once you’ve installed it, just log in, select a date-range (week) and a query and start logging time.

It’s free for up to 5 users so you can try it out to see if it works for you and your organization. There are some report samples out-the-box and you can also easily create your own reports using PowerPivot.

We used Entity Framework, MVC, Bootstrap and Knockout to make the site. Most of our JavaScript is typed using TypeScript. Of course we have unit tests (.NET and JavaScript) and a build that builds the installer (Wix) package. It was a fun project to work on and I think we’ve turned out a great product. Download your copy today!

Here’s the overview video:

Happy timetracking!

