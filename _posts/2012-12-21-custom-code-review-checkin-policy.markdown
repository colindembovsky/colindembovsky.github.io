---
layout: post
title: Custom Code Review Checkin Policy
date: '2012-12-21 07:33:00'
tags:
- alm
---

Over the last couple of months, I’ve been doing a lot of VS / TFS 2012 demos, as well as installing / configuring / customizing TFS at customers. Everyone loves the Code Review Workflow, but inevitably the question gets asked, “Can I enforce code review on check-in”? Out of the box you can’t, but I created a custom policy for this.

You can get the policy from the [VS Gallery](http://visualstudiogallery.msdn.microsoft.com/c476b708-77a8-4065-b9d0-919ab688f078).

Note: this policy only works with “out of the box” Code Review Request and Response work item types in TFS 2012 and for VS 2012.

You can also configure how “strict” the review policy should be:

- The policy will fail if the Code Review Request is not Closed
- The policy will fail if any response result is 'Needs Work'
- The policy will pass if all response results are 'Looks Good' or
- the policy will pass if all response results are 'Looks Good' or 'With Comments'
<figure class="kg-card kg-image-card"><img src="http://i1.visualstudiogallery.msdn.s-msft.com/c476b708-77a8-4065-b9d0-919ab688f078/image/file/91116/1/capture.png" class="kg-image" alt loading="lazy"></figure>

If the policy fails, you’ll get a friendly message reminding you to get a review!

<!--kg-card-begin: html--> [![image](http://lh4.ggpht.com/-4zufP2hpTXg/UNOEC2eJ49I/AAAAAAAAAiQ/v4KUoVUk6Y4/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-bK5Hs1G5t2s/UNOEBvcXJtI/AAAAAAAAAiI/P1EkUhJXmkw/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

Happy reviewing!

