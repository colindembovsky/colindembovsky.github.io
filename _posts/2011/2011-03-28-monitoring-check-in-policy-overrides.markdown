---
layout: post
title: Monitoring Check-in Policy Overrides
date: '2011-03-28 16:06:00'
tags:
- alm
---

I’m often asked by customers why there is no way to prevent check-in policy overrides in TFS. Usually I respond along the lines of, “Well, it should be a really rare occurrence (otherwise why have the policy?) and besides, every action against TFS is tracked, so you can monitor overrides and beat ‘chat nicely’ to the developer who is overriding the policies”. Which is all well and fine, but exactly _how do you monitor policy overrides_?

This weekend I checked Amazon to see if I could find [Professional Team Foundation Server 2010](http://www.amazon.com/Professional-Team-Foundation-Server-ebook/dp/B004S82RRE/ref=sr_1_1?ie=UTF8&m=AG56TWVU5XWC2&s=books&qid=1301295041&sr=8-1) for my Kindle – and to my surprise, there it was! [Ed Blankenship](http://www.edsquared.com/), one of the authors, is a colleague at [Notion Solutions](http://www.notionsolutions.com/) and is an absolute genius on all things TFS. The other authors are, of course, TFS legends too! I instantly purchased the book and while going through it, found a section in Chapter 7 that speaks about monitoring check-in policy overrides.

There are 2 main ways of doing so – via email alerts and via the warehouse.

## Email Notification

Install the latest version of the [Team Foundation Power Tools](http://visualstudiogallery.msdn.microsoft.com/c255a1e4-04ba-4f68-8f4e-cd473d6b971f/) – anyone using TFS should always have this installed! One of the features that gets installed with these tools is a richer notifications manger – the Alerts Explorer.

In VS, go to Team-\>Alerts Explorer and add a “Check-In to specific folder with policy overriden” alert (under the Check-in alerts section).

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/_d41Ixos7YsM/TZAzWbl2ASI/AAAAAAAAAQY/Y4s1g_mtKPc/image_thumb.png?imgmax=800 "image")](http://lh6.ggpht.com/_d41Ixos7YsM/TZAzVMbFlkI/AAAAAAAAAQU/BltKpBs28gU/s1600-h/image%5B2%5D.png)<!--kg-card-end: html-->

Now you’ll get an email whenever someone does something nefarious and overrides the check-in policy you’ve so meticulously planned and put in place…

## Monitoring via the Warehouse

The second method of monitoring check-in overrides is to query the warehouse. In the warehouse database (called Tfs\_Warehouse by default) there is a table called

<!--kg-card-begin: html--><font face="Courier New">DimChangeset</font><!--kg-card-end: html-->

. One of the columns on this table is

<!--kg-card-begin: html--><font face="Courier New">PolicyOverrideComment</font><!--kg-card-end: html-->

, which is null whenever there is no override and which contains the policy override comment (which is mandatory when overriding) otherwise. So you can easily craft a query (and turn it into a report) that looks for check-ins with a

<!--kg-card-begin: html--><font face="Courier New">PolicyOverrideComment</font><!--kg-card-end: html-->

. Here’s the T-SQL:

    <span style="color: blue">SELECT<br> </span><span style="color: teal">[ChangesetSK]</span><span style="color: gray">,<br> </span><span style="color: teal">[ChangesetID]</span><span style="color: gray">,<br> </span><span style="color: teal">[ChangesetTitle]</span><span style="color: gray">,<br> </span><span style="color: teal">[PolicyOverrideComment]</span><span style="color: gray">,<br> </span><span style="color: teal">[LastUpdatedDateTime]</span><span style="color: gray">,<br> </span><span style="color: teal">[TeamProjectCollectionSK]</span><span style="color: gray">,<br> </span><span style="color: teal">[CheckedInBySK]<br></span><span style="color: blue">FROM<br> </span><span style="color: teal">[Tfs_Warehouse]</span><span style="color: gray">.</span><span style="color: teal">[dbo]</span><span style="color: gray">.</span><span style="color: teal">[DimChangeset]<br></span><span style="color: blue">WHERE<br> </span><span style="color: teal">[PolicyOverrideComment] </span><span style="color: gray">IS NOT NULL<br></span>

Happy monitoring!

