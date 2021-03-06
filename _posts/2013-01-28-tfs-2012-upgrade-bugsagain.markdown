---
layout: post
title: TFS 2012 Upgrade Bugs–again?
date: '2013-01-28 21:17:00'
tags:
- alm
---

On January 11, [Brian Harry blogged](http://blogs.msdn.com/b/bharry/archive/2013/01/11/tfs-2012-update-1-hotfix.aspx) about some bugs in the TFS 2012 upgrade process (as well as the link to the KB patch for fixing the bugs). I was upgrading a customer in December last year and we hit one of the “symptoms” that was fixed with the patch.

I was at another customer about two weeks ago and we applied the patch before importing Team Project Collections and of course we didn’t hit any of the bugs. It was a great week for me!

Unfortunately, I didn’t have a great week last week when I went to upgrade another customer from TFS 2010 to TFS 2012. I applied the patch from Brian’s blog and while the Team Project Collections imported successfully, some other very strange things started to happen.

Most of the “bugs” occur around security – not being able to see collections that you have access to (or seeing collections you don’t have access to) for example. But we also had some obscure issues that were also security related, but were harder to diagnose.

For example, we had 22 Test Plans, and suddenly 4 of them were “broken”. If you tried to connect to these Test Plans, a popup appeared saying, “Team Foundation Server is currently offline. Try again later”. This of course wasn’t the case, since the server was up and running and we could access 18 other Test Plans! The event logs on the TFS server had an exception “Given key was not present in the dictionary”.

We also noticed that if you accesses a Test Plan that wasn’t “broken” and went to the Results pane, you got an “Object reference not set to an instance of an object” error message. Not cool…

Fortunately, being an MVP has its advantages. I mailed the champs list and within a couple of hours, members of the product team were helping me. Eventually they provided me with a script to run against the TFS database that sorted the weird issues, and my customer was able to get back to work. In fact, when the team provided me the script they told me that they had already found this bug and were including it in a second patch for the upgrade process. I don’t have a timeline on when this patch will be released – but I am confident that the product team has it in hand!

## Unofficial Analysis

I through back to why out of the three upgrades I’ve done, two seemed to have issues while the 3rd was error free. In both cases where there were issues, changes had been made to users on the AD domain that the TFS server was on – users had been deleted after they had touched the old TFS. I suspect that something in the identities logic wasn’t catering for “missing” users when you install TFS and then import a Team Project Collection that has “old” users.

## Quarterly Pain?

I love the fact that TFS on-premise is going to a quarterly release cycle. This is good news, especially for those of us who like new features and who like to be at the forefront. However, I hope that the product team improves the upgrade process. Of course this process is mammoth, and the number of permutations they have to cater for is staggering, so I’m not saying it’s going to be a cakewalk. However, if we’re going to be eager for four updates a year, we need to feel confident that they’ll “just work”.

A big thank you to Chandru Ramakrishnan and his fellows for the speedy help!

Happy upgrading!

