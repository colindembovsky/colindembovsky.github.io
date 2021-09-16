---
layout: post
title: Branch Info Team Explorer Extension (BITE) Now Available for VS 2013
date: '2013-08-31 03:24:00'
tags:
- tfsapi
- alm
---

 **<u>Update 2013-09-12:</u>** I’ve updated the [extension](http://visualstudiogallery.msdn.microsoft.com/33a5274b-271b-45dd-8dc5-734d927a19dd) to work with VS 2013 RC (since there were some breaking changes from Preview).

I wrote a Team Explorer Extension ([BITE](http://visualstudiogallery.msdn.microsoft.com/1d61464c-65af-4d25-af15-3b6b6919c56e)) [a few months ago](http://www.colinsalmcorner.com/2013/07/bitebranch-info-team-explorer-extension.html) to show you which branch your solution is on and how to easily change to the same solution on another branch.

Today I opened up and converted the extension for [VS 2013](http://visualstudiogallery.msdn.microsoft.com/85185516-dfe6-44e6-aa64-892cbff0e98a).

Since the architecture of the Home page in Team Explorer has changed a little, it wasn’t simply open and recompile for VS 2013. The BITE page actually stayed the same – but the classes that allow me to hook into the Team Explorer had to change quite a lot. And of course there’s scant documentation – even for extending 2012, never mind the dearth of information about extending Team Explorer 2013. Anyway, nothing that Reflector couldn’t help me with…

Here’s what the extension looks like in the VS 2013 Team Explorer Home page:

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-hkEo-H5clLM/UiDjZwDMIWI/AAAAAAAABDc/7NNclLymTdA/image_thumb.png?imgmax=800 "image")](http://lh5.ggpht.com/-_cFZK458Psc/UiDjZPBkdgI/AAAAAAAABDU/_By03NAdx18/s1600-h/image%25255B2%25255D.png)<!--kg-card-end: html-->

While I was at it, I cleaned up the UI a little especially when you don’t have a solution open or the solution you have open is not branched. Other than that, it works exactly like before – only for TFVC, of course!

Happy branching!

