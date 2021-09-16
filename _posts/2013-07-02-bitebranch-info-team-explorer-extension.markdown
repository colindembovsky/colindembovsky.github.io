---
layout: post
title: BITE–Branch Info Team Explorer Extension
date: '2013-07-02 15:38:00'
tags:
- development
- sourcecontrol
---

Update 2013-08-30: The extension is now available for [VS 2013](http://visualstudiogallery.msdn.microsoft.com/85185516-dfe6-44e6-aa64-892cbff0e98a).

Branching is something that you should definitely be doing if you’re a modern developer. It doesn’t matter if you have branch-per-release or dev-main-live kind of branching – you need to be able to separate development streams.

So let’s pick on the classic dev-main-live scenario. You create a solution in Visual Studio (in a MAIN folder), check it into Source Control in TFS and then create branches to DEV and LIVE. Now you have three versions of the same solution – one on each branch. I always recommend opening from Source Control so that you know which branch you’re on. However, if you’ve opened the solution and then been working for a while, you may want to double-check which branch you’re working from. Hmmm, you’re stuck – there’s no way to do this other than checking the folder path for the solution.

## Enter BITE

Wouldn’t it be cool if you could instantly see which branch your solution is on? And how about selecting one of the other branches and clicking a “Switch” button to switch to the same solution on another branch? Now you can – using BITE – the Branch Info Team Explorer Extension! (Yes, I know it’s cheesy, but I couldn’t help it once I’d seen the acronym).

Once you’ve installed the extension from the [VS Gallery](http://visualstudiogallery.msdn.microsoft.com/1d61464c-65af-4d25-af15-3b6b6919c56e), you’ll see a new Link under the Pending Changes section:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-cFPhFgEFITg/UdJ1WvjRyaI/AAAAAAAAA8o/cDryt8aM4vI/image_thumb.png?imgmax=800 "image")](http://lh4.ggpht.com/-gvgJxO3NjxI/UdJ1VaPQlvI/AAAAAAAAA8g/Z1aLvetvQhU/s1600-h/image2.png)<!--kg-card-end: html-->

Click on “Branch Info” to see the extension in action.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-CpT7_b_rqPc/UdJ1YKi0eRI/AAAAAAAAA84/iCklL1PlQhs/image_thumb3.png?imgmax=800 "image")](http://lh6.ggpht.com/-UeRKHCH7Lk4/UdJ1XMB_rUI/AAAAAAAAA8w/Si3T8G_ARZU/s1600-h/image9.png)<!--kg-card-end: html-->

Here I’ve opened a solution on the MAIN branch (see the Current Branch label). I can see both the local and server paths for the solution. Also, there’s a drop-down labelled “Other Branches”. If I select one of the other branches, I can click the “Switch” button and the corresponding solution opens.

Let me know if you have any issues using this extension. (In case you missed the link, get the extension from the [VS Gallery](http://visualstudiogallery.msdn.microsoft.com/1d61464c-65af-4d25-af15-3b6b6919c56e)).

Happy branching!

