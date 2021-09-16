---
layout: post
title: Enable Custom IntelliTrace Web Events with a Right-Click
date: '2013-04-12 15:15:00'
tags:
- news
- development
---

Series Links:

- [Part 1](http://www.colinsalmcorner.com/2013/06/intellitrace-tips-and-tricks-basics.html): The Basics (this post) – also [guest posted](http://blogs.msdn.com/b/southafrica/archive/2013/05/13/guest-post-intellitrace-tips-and-tricks-the-basics-part-1-colin-dembovsky.aspx) on MSDevDiv SA Blog
- [Part 2](http://www.colinsalmcorner.com/2013/06/intellitrace-tips-and-tricks.html): IntelliTrace Everywhere – also [guest posted](http://blogs.msdn.com/b/southafrica/archive/2013/05/13/guest-post-intellitrace-tips-and-tricks-intellitrace-everywhere-part-2-colin-dembovsky.aspx) on MSDevDiv SA Blog
- [Part 3](http://www.colinsalmcorner.com/2013/04/enable-custom-intellitrace-web-events.html): Enable Custom IntelliTrace Events with a Right-Click

**Note: This is an unsupported feature!** Use at your own risk – though to be honest I can’t see what the risk really is. Just know that this is not supported by Microsoft.

## Preamble (TL;DR – skip to next section for the Good Stuff)

I’m going to be presenting 3 deep dives at [TechEd Africa](http://www.teched.co.za/) next week:

- Version Control ([including Git](http://blogs.msdn.com/b/bharry/archive/2012/08/13/announcing-git-integration-with-tfs.aspx))
- [Agile Planning Tools](http://blogs.msdn.com/b/bharry/archive/2013/04/04/vs-tfs-2012-2-update-2-released-today.aspx) and Customizations
- [IntelliTrace](http://msdn.microsoft.com/en-us/library/vstudio/dd264915.aspx)

While I was preparing for my IntelliTrace session, I was watching [Larry Guger](http://continuouslyintegrating.blogspot.com/) (IntelliTrace PM) do a presentation when [IntelliTrace-in-Production](http://msdn.microsoft.com/en-us/library/vstudio/hh398365.aspx) had just launched. Right at the end of his talk, he did something which boggled my mind – he right-clicked a method and right there in the context menu was “Insert IntelliTrace Event”. He did this for a couple of methods, and then exported these events to a folder. When he ran the IntelliTrace collector for IIS using PowerShell, the custom events were present in the log file.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-tP5bj2cs4-k/UWemLeFP79I/AAAAAAAAArU/7zJG4RSFRoM/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-dvLBpOnAXds/UWemJ36qb5I/AAAAAAAAArM/uvPWeP3hZEQ/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

This was amazing because I know how tedious it is to create custom IntelliTrace events. I quickly fired up my VS just to see if I could do it – and couldn’t find the menu option.

I then mailed Larry and after a short email conversation and some scratching around the IntelliTrace dll’s with Reflector, I was able to figure out that if you add a registry entry, you “unlock” this feature.

This is a “partially complete” feature in VS – here is the big limitation:

**This only works with IntelliTrace collection for IIS applications via PowerShell**.

## The Good Stuff

So I’ll cut to the chase: here’s the registry key:

<!--kg-card-begin: html--><font face="Courier New">Windows Registry Editor Version 5.00</font><!--kg-card-end: html--><!--kg-card-begin: html--><font face="Courier New">[HKEY_CURRENT_USER\Software\Microsoft\VisualStudio\11.0_Config\TraceDebugger]<br>"IntelliTraceEventsEnabled"=dword:00000001</font><!--kg-card-end: html-->

Just copy and paste this into a file (intelli.reg or something) and double-click it. Restart VS.

Now open up a Web Application, find a method, right click, select “Insert IntelliTrace Event”. Repeat for a couple of other methods. You’ll see a glyph in the gutter indicating that you have an event for that method.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-DcRv07DyD5s/UWemNh-msOI/AAAAAAAAArk/cwTHy1vpNm0/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-CENUNMpO-cE/UWemMlMqYXI/AAAAAAAAArc/WyaWXJoAdLk/s1600-h/image%25255B7%25255D.png)<!--kg-card-end: html-->

Now go to Debug-\>IntelliTrace-\>Export IntelliTrace Events. Save this file to a folder somewhere.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-aKHozhN87bw/UWemRKDkOYI/AAAAAAAAAr0/f2mO0X2b_Co/image_thumb%25255B6%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-EiEhnhRSQhU/UWemPGNY9iI/AAAAAAAAArs/6f6hZsPLM-g/s1600-h/image%25255B12%25255D.png)<!--kg-card-end: html-->

You’ll see that the file has an .iFragment extension – this is an IntelliTrace config fragment. If you open up the file, you’ll see it has created Category, Module and Diagnostic sections – these are the same sections you’d have to manually create if you wanted some custom IntelliTrace events.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-U-40H7ZwnqI/UWemTmhaNlI/AAAAAAAAAsE/7NMY9idTF4Q/image_thumb%25255B14%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-DhL5tFzpOrg/UWemR5PRDoI/AAAAAAAAAr8/Nd1IckmzTtk/s1600-h/image%25255B26%25255D.png)<!--kg-card-end: html-->

**Aside:** You can see the “AutomaticDataQuery” element in the &nbsp;tag. This allows you to see the in/out arguments of the method in the locals window when you debug the log file later… No messing around with argument positions and stuff…

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-MAt8013SHKc/UWemWK4iY-I/AAAAAAAAAsU/qpFiVWhICdU/image_thumb%25255B16%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-DxpfSF2FsRc/UWemUrhzS5I/AAAAAAAAAsM/IP7f3a3NdK0/s1600-h/image%25255B30%25255D.png)<!--kg-card-end: html-->
## Using iFragments

Now that you have a custom iFragment, you need to go to the server that you want to collect IntelliTrace events from. Go to the folder where you extracted the collector and open (or create) a folder called **CustomEvents**. This is where you drop your iFragments.

Now fire up your collector (using the default collection plan). Collect a log. Open it in VS. Start debugging.

The first thing to note is that you have a custom category in the categories list of the IntelliTrace events window:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-C7wqpOBQ_ho/UWemY_oC4kI/AAAAAAAAAsk/T8OIUaZcO_Y/image_thumb%25255B10%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-DjF5GKXphvk/UWemXeipbvI/AAAAAAAAAsc/FpGjwQJ20uA/s1600-h/image%25255B18%25255D.png)<!--kg-card-end: html-->

Secondly, you’ll see the events by the same glyph that you saw when you added the events in VS:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-LP0JJZS-vMI/UWemcJkbMmI/AAAAAAAAAs0/lHKQKKfeltk/image_thumb%25255B12%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-t-R1fkao-sY/UWemaIzjMNI/AAAAAAAAAss/3NR52N_L2GE/s1600-h/image%25255B22%25255D.png)<!--kg-card-end: html-->

Happy logging!

