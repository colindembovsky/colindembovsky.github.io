---
layout: post
title: 'IntelliTrace Tips and Tricks: The Basics'
date: '2013-06-18 21:33:00'
tags:
- development
---

Series Links:

- [Part 1](http://www.colinsalmcorner.com/2013/06/intellitrace-tips-and-tricks-basics.html): The Basics (this post) – also [guest posted](http://blogs.msdn.com/b/southafrica/archive/2013/05/13/guest-post-intellitrace-tips-and-tricks-the-basics-part-1-colin-dembovsky.aspx) on MSDevDiv SA Blog
- [Part 2](http://www.colinsalmcorner.com/2013/06/intellitrace-tips-and-tricks.html): IntelliTrace Everywhere – also [guest posted](http://blogs.msdn.com/b/southafrica/archive/2013/05/13/guest-post-intellitrace-tips-and-tricks-intellitrace-everywhere-part-2-colin-dembovsky.aspx) on MSDevDiv SA Blog
- [Part 3](http://www.colinsalmcorner.com/2013/04/enable-custom-intellitrace-web-events.html): Enable Custom IntelliTrace Events with a Right-Click

This brief series of blog posts will show you how to get the most out of IntelliTrace – a historical debugger that allows you to record (and replay) program execution.

## Using IntelliTrace

You can use IntelliTrace in the following scenarios:

1. During debugging in VS using F5
2. In Test Manager (enabled as a Data Diagnostic Adapter)
3. Anywhere (using the standalone collector)

## Understanding IntelliTrace “Modes”

Out of the box, IntelliTrace has 2 broad modes – “Events Only” and “Events and Call Information”. Events Only is much more lightweight, and allows you to record “events” that occur when your program is running. These “events” are “interesting occurrences” – like ASP.NET calls or ADO.NET calls – or exceptions. The product team tried to record events which would help you understand how your code works so that you can understand long “cause and effect” chains.

Events and Call Information turns the dial up a lot – it collects not only all of the events from the Events Only mode, but also method calls (including in and out arguments). There are some limits placed on what data is collected – otherwise the logs would become even more enormous than they are using the default settings.

## F5 IntelliTrace

Use this when you’re coding. It means you can debug and move back and forwards in your debug session without having to stop your application and add breakpoints. To turn it on, go to Tools-\>Options and go to IntelliTrace settings. Here you’ll see the 2 modes:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/--zXqZig5Sf8/UcBTIkP0YzI/AAAAAAAAA4M/1v74jQp6eCA/image_thumb11.png?imgmax=800 "image")](http://lh6.ggpht.com/-CYt5QEthRTc/UcBTHGhxisI/AAAAAAAAA4E/7jIrJmB1AUg/s1600-h/image3.png)<!--kg-card-end: html-->

Once you’ve turned it on (in this case I’ve got the Events and Call Information mode on) you can start debugging. Click around your app, and then click the “Break All” link in VS in the IntelliTrace window:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-EGHkvfBc-uA/UcBTLe54PAI/AAAAAAAAA4c/lC_QonAhUms/image_thumb31.png?imgmax=800 "image")](http://lh6.ggpht.com/-PwYyK9Zowug/UcBTKOpZYnI/AAAAAAAAA4U/RsA_8d-Xb-8/s1600-h/image7.png)<!--kg-card-end: html-->

(Here I am debugging my Calculator application – click “Break All” to go to the IntelliTrace log).

When you look at the log, you’ll initially see all the events that occurred. In this case, button clicks are “Gesture” events, so that’s what I see:

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-8pLWIk-38mI/UcBTN6AE2jI/AAAAAAAAA4s/L_T2kYa3rDE/image_thumb5.png?imgmax=800 "image")](http://lh3.ggpht.com/-68iYP0exLcU/UcBTMcXJACI/AAAAAAAAA4k/yEgRe86RtBE/s1600-h/image11.png)<!--kg-card-end: html-->

Let’s say that I remember something weird happened when I clicked “=” when multiplying 2 numbers – well, I don’t have to restart the application, I can simply click on the “=” gesture after the “\*” button click (in the above log, I’ve clicked 3 x 6 =). So I can click on that event and “rewind” to that point.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-1qVinYo0zzY/UcBTQKZrq0I/AAAAAAAAA48/ZPUovcn1Fsw/image_thumb6.png?imgmax=800 "image")](http://lh5.ggpht.com/-A-3Uofqd3y0/UcBTO2c5P4I/AAAAAAAAA40/grT2xnpLDOU/s1600-h/image14.png)<!--kg-card-end: html-->

What I can now do is click “Calls View” to start walking through the log from that point.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-a_Qc-RhXtRM/UcBTTLKR2eI/AAAAAAAAA5M/nsEyxmInQOQ/image_thumb8.png?imgmax=800 "image")](http://lh6.ggpht.com/-DaLsuX2tB3I/UcBTRZ1I3aI/AAAAAAAAA5E/qMOfr1Ot6oA/s1600-h/image18.png)<!--kg-card-end: html-->

Double-click on the call to btnEqual\_Click to “zoom” to that point of the execution. You’ll see the IntelliTrace glyphs in the gutter of the source window:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-jBH7H6YzVHI/UcBTVrNDPSI/AAAAAAAAA5c/0J_w722KdPU/image_thumb111.png?imgmax=800 "image")](http://lh6.ggpht.com/-Kfln_SXOdlc/UcBTUNbxSII/AAAAAAAAA5U/KJ8Jc6oBOsw/s1600-h/image41.png)<!--kg-card-end: html-->

If you mouse-over the icons, you’ll get a tooltip. The functions (in the order they appear) are:

- return to calling method
- step back
- step into
- step over
- return to Live Debugging

(Just a handy tip: If you press the bottom button “return to Live Debugging” remember that this now puts you at the current debugger point – you’ll need to press F5 again if you want to start running the application again.)

If you advance using F11, the current pointer will advance. Pressing twice from the previous image gets me to the point where the program enters the switch statement:

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-3yu_4eE8pLw/UcBTYVFG5UI/AAAAAAAAA5s/mcRV54X6YtU/image_thumb3.png?imgmax=800 "image")](http://lh3.ggpht.com/-lero369BGZo/UcBTXPpGgzI/AAAAAAAAA5k/mBIc2JBDELw/s1600-h/image8.png)<!--kg-card-end: html-->

Notice that I don’t need to guess which case statement was selected – the log records exactly what the program did. I’ve also pinned the mouse-overs for val1 and val2 – you can see their values. Not all values are collected, but since these are primitives and are being passed into (or out of) a method, IntelliTrace dutifully collects their values.

I’ll press F11 again to step into the Multiply() method, and then F11 again to advance one more event:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-Y6Maui4gC_c/UcBTaidU2kI/AAAAAAAAA58/Gy7rtaP-3Sg/image_thumb51.png?imgmax=800 "image")](http://lh6.ggpht.com/-AHFOiZK2Tgo/UcBTZSq8ToI/AAAAAAAAA50/nUiREd8Zdpo/s1600-h/image12.png)<!--kg-card-end: html-->

That puts me onto the closing curly brace of the Multiply method. If I look in the Autos window, I’ll see that the method in parameters (val1 and val2) were 3 and 6 respectively, and that the return value was 9.

Voila – with 3 or 4 clicks we found the bug in our program. And we didn’t even have to set a breakpoint!

## Filtering Events

As you use IntelliTrace, you’ll start seeing large amount of events – to make the logs easier to navigate, you can use the category, thread and search box at the top of the events window to filter the events.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-NtpzfhfOtrE/UcBTdEzj4TI/AAAAAAAAA6M/wKWY2GC2UCE/image_thumb6%25255B1%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-CkpyfLc3cYE/UcBTbg5zUJI/AAAAAAAAA6E/kPFhUlPrTuc/s1600-h/image13.png)<!--kg-card-end: html-->

For example, expanding the dropdown with “All Categories” in it I can filter just exceptions:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-5m_cz6gQRyg/UcBTgegBALI/AAAAAAAAA6c/MXjyjFuQ5sg/image_thumb8%25255B1%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-aWhGA-AlR_E/UcBTe3efEwI/AAAAAAAAA6U/QNxzunCeLvU/s1600-h/image17.png)<!--kg-card-end: html-->
## Search For This Line…

Another useful tip is the “Search For this line” or “Search For This Method”. You suspect that a method was hit sometime during your debug session – but the log is quite long. No problem – right click a line (or method) and select “Search For This Line / Method in IntelliTrace”.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/--HZ_0Wa_Au8/UcBTjbnE8qI/AAAAAAAAA6s/WG4zkUOlERA/image_thumb1.png?imgmax=800 "image")](http://lh5.ggpht.com/-PmAywdKi4t4/UcBThc9D6zI/AAAAAAAAA6k/Ll5EfRwKTBA/s1600-h/image411.png)<!--kg-card-end: html-->

Once you do, you’ll see the search results in a bar at the top of the current code window – press the arrows to go to the previous or next instance of that line or method in the log file:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-ThnqArcP5nY/UcBTnP8xqoI/AAAAAAAAA68/yRVA9c_eUx8/image_thumb9.png?imgmax=800 "image")](http://lh5.ggpht.com/-NSCwg7KR6Xg/UcBTkvgC48I/AAAAAAAAA60/ccOa7YzVC38/s1600-h/image18%25255B1%25255D.png)<!--kg-card-end: html-->

Here I’ll click on the arrow icon directly after the work “Multiply” to go to the first call to this method in the log, and I can start “debugging” from there.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-RwFRQkuZIiU/UcBTsImCP0I/AAAAAAAAA7M/EII8TwZlhNQ/image_thumb4.png?imgmax=800 "image")](http://lh5.ggpht.com/-sA8MQhwTELo/UcBTqCo87bI/AAAAAAAAA7E/RumDaU3mTvg/s1600-h/image9.png)<!--kg-card-end: html-->

In the next post, I’ll show you how to run IntelliTrace anywhere and everywhere.

Happy debugging!

