---
layout: post
title: 'Fix: You Open TfcvTemplate.12.xaml and Don’t See Any Parameters'
date: '2013-09-14 00:23:00'
tags:
- build
---

I upgraded my demo environment from 2013 Preview to 2013 RC. Everything looked good until I got to the builds. I had configured a couple of default builds – the 2013 default template is actually stored in the TFS database (not in source control like the old Default xaml files) unless you actually download it for customizing.

However, when I opened the build definition, the parameters section of the Process tab was empty:

<!--kg-card-begin: html-->[![clip_image001](http://lh4.ggpht.com/-wZkhv8yNy_w/UjMt6Hy0ZpI/AAAAAAAABFI/a6TAabG9fXI/clip_image001_thumb%25255B1%25255D.jpg?imgmax=800 "clip\_image001")](http://lh4.ggpht.com/-xpu4oH2llnU/UjMt5Ek83oI/AAAAAAAABFA/zBq_rr_Uj0Q/s1600-h/clip_image001%25255B4%25255D.jpg)<!--kg-card-end: html-->

All the other tabs worked just fine, and all the other (XAML from source control) templates worked just fine too.

I quickly mailed the Champs List, and got some great assistance from Jason Prickett of the product team. I attached a debugger to VS and opened the template, and I got some “could not load assembly” errors for Newtonsoft.Json.dll.

Jason then told me the solution was simple: copy the dll. So I copied

<!--kg-card-begin: html--><font size="2" face="Courier New">C:\Program Files\Microsoft Team Foundation Server 12.0\Tools\Newtonsoft.Json.dll</font><!--kg-card-end: html-->

to

<!--kg-card-begin: html--><font size="2" face="Courier New">C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\IDE\PrivateAssemblies\</font><!--kg-card-end: html-->

and that resolved the problem.

Now I can create, edit and run builds again. And I’m loving the [new RC features](http://nakedalm.com/whats-new-in-visual-studio-2013-rc-with-team-foundation-server/).

Happy building!

