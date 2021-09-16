---
layout: post
title: Using Indexed Published Symbols from TeamBuilds
date: '2011-02-08 15:05:00'
tags:
- build
---

You’ve seen the indexing and symbol publishing options when you set up a Team Build – but have you ever tried to debug and application that has symbols? How do you get VS to use the published symbols? What are these settings for anyway?  
Source indexing and published symbols make possible to debug an application without shipping symbols. Source indexing produces pdb files during a build – essentially mapping the binary to the source code. During a team build, extra information is wrapped inside the pdbs – like the Team Foundation Server URL where they were built and version control paths to and versions of source code. This allows VS to download source code for debugging – and since there’s security involved (to access the TFS server) – source server support is disabled for debugging by default.  
Fortunately enabling it is quite easy – especially if you read your Kindle copy of Inside the [MS Build Engine (2nd edition)](http://www.amazon.com/gp/product/0735645248/ref=s9_simh_gw_p14_d5_i4?pf_rd_m=ATVPDKIKX0DER&pf_rd_s=center-2&pf_rd_r=19SEA1DDD7X2GG6BYFQM&pf_rd_t=101&pf_rd_p=470938631&pf_rd_i=507846) on the plane!  
Open VS and go to Tools-\>Options-\>Debugging-\>General settings and tick the “Enable source server support”.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/_d41Ixos7YsM/TVDO68P984I/AAAAAAAAANk/bWlYznlrpvE/image_thumb4.png?imgmax=800 "image")](http://lh3.ggpht.com/_d41Ixos7YsM/TVDO38WDEsI/AAAAAAAAANg/USzGOo6YjW0/s1600-h/image8.png)<!--kg-card-end: html-->

The DefaultTemplate.xaml for default builds supports symbol publishing. When you set up a Team Build, expand the “Basic Section”, set “Index Sources” to True and enter a UNC for the “Path to Publish Symbols” – try to stick to one global symbol store – TeamBuild will organise the symbols within this folder. The only time you’d really want multiple stores is for concurrency – only one build can publish at a time (that’s the way the Default Template is designed), so if you have lots of build going all the time and the indexing is slowing them, then you may want to add another store or two.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/_d41Ixos7YsM/TVDO-DrempI/AAAAAAAAANs/l0rSBypW9Uo/image_thumb%5B6%5D.png?imgmax=800 "image")](http://lh3.ggpht.com/_d41Ixos7YsM/TVDO8F4OKCI/AAAAAAAAANo/Yt-KZRQ6s30/s1600-h/image%5B6%5D.png)<!--kg-card-end: html-->

The final step is to tell VS where to find the published symbols. To add a symbol store in VS, go to Tools-\>Options-\>Debugging-\>Symbols and add the UNC to the store.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/_d41Ixos7YsM/TVDPDm1wNGI/AAAAAAAAAN0/F7Y96aDQKTg/image_thumb3.png?imgmax=800 "image")](http://lh6.ggpht.com/_d41Ixos7YsM/TVDPA_RWrMI/AAAAAAAAANw/zCxbvkF-Geg/s1600-h/image7.png)<!--kg-card-end: html-->

Happy debugging!

