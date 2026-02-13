---
layout: post
title: Azure Outage – I was a victim too, dear Reader
date: '2014-11-21 22:35:02'
tags:
- news
---

This morning I went to check on my blog – the very blog you’re busy reading – and I was greeted with a dreaded YSOD (Yellow Screen of Death). What? That can’t be! I haven’t deployed anything since about 10 days ago, so I know it wasn’t my code! What gives?

It turns out that I had some garbled post files. My blog is built on [MiniBlog](https://github.com/madskristensen/MiniBlog) which stores all posts as xml files. One of the changes I made to my engine is to increment a view counter on each post so that I can track which posts are being hit. I suppose there is some risk in doing this, since there is a lot of writing to the files. Turns out about 6 files in my posts directory were either empty or partially empty – I suspect that IIS was writing the files when [the outage happened a couple of days ago](http://azure.microsoft.com/blog/2014/11/19/update-on-azure-storage-service-interruption/). Anyway, turns out MiniBlog isn’t that resilient when the xml files it’s reading are not well-formed xml! That just goes to show you that even though your code has been stable for months, stuff can still go wrong!

So I applied a fix (which moves the broken file out the way and sends me an email with the exception information) and it looks like the site is up again. Although at time of writing, the site is dog slow and I can’t seem to WebDeploy – I’ve had to use FTP to fix the posts and update my code, and I can’t see the files using the Azure SDK File Explorer (though I am able to see the logs, fortunately). I suspect there are still Azure infrastructure problems.

So other than having my counts go wonky, I may have lost a couple of comments. If one of your comments disappeared, Dear Reader, I humbly apologize.

Also, this is my first post from my Surface 3 Pro! Woot!

Happy reading-as-long-as-Azure-stays-up!

