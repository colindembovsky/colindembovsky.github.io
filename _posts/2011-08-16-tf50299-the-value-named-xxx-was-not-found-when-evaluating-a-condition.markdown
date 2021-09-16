---
layout: post
title: 'TF50299: The value named ‘xxx’ was not found when evaluating a condition'
date: '2011-08-16 04:31:00'
tags:
- tfsconfig
---

Recently while working at a customer, we configured mail alerts for TFS. We checked that the SMTP server was correct and that we could send mail from the application tiers – everything looked correct, but still there were no mails.

In the event log, we found this “helpful” error (note the sarcasm):

<!--kg-card-begin: html--><font face="Courier New">ResultMessage : Job extension had an unhandled error: System.Exception: TF50299: The value named '070001' was not found when evaluating a condition.</font><!--kg-card-end: html-->

Unfortunately, there was woefully little information about this error.

The Team Project Collection was upgraded from a TFS2008 server, so it inherited a lot of “old” notifications. We eventually scanned the notifications, and found one that had a rather strange clause in it:

<!--kg-card-begin: html--><font face="Courier New">… AND ‘070001’ = ‘MyProject’…</font><!--kg-card-end: html-->

It seemed strange to me that there appeared to be value parameters on the left and the right of the = operator – more usual conditions look like

<!--kg-card-begin: html--><font face="Courier New">… AND TeamProject = ‘MyProject’…</font><!--kg-card-end: html-->

So we deleted this notification, and suddenly all the mails started coming through. It seems like this exception crashes the notification job completely – it would be nice if it just skipped this notification clause and then continued with the other notifications!

Moral of the story – make sure your notifications have properly formed condition clauses!

