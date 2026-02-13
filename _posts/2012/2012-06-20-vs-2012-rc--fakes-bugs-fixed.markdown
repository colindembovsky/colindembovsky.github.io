---
layout: post
title: VS 2012 RC – Fakes Bugs Fixed
date: '2012-06-20 19:27:00'
tags:
- development
---

In a [previous post](http://colinsalmcorner.blogspot.com/2012/04/more-on-fakes-beta-has-issues.html) about the MS Fakes framework, I made mention of some bugs that appeared in the Beta. I finally had some time to test out the same code in the RC, and I am pleased to tell you that the bugs have been fixed (well, the ones I found anyway!).

## Upgrading Gotcha

The only gotcha I came across appears to be a snag when you open in the RC a test project that was created in the Beta. I initially simply opened up the code that I had from the Beta, updated references to TeamFoundation dlls from version 10 to version 11, regenerated the fakes and tried to run the tests. FAIL – the tests were failing with “ShimNotSupportedExceptions”. After trying various things, I noticed some text in the Output window from the “Tests” section (the Show output from dropdown):

<!--kg-card-begin: html--><font size="3" face="Courier New"> <p></p>
<pre>Failed to configure settings for runsettings plugin ‘Fakes’ as it threw following exception:<br>‘Object reference not set to an instance of an object’<br>Please contact the plugin author.<br></pre></font><!--kg-card-end: html-->

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-XX9_uZrD4ZA/T-GlgTnpT-I/AAAAAAAAAY4/dFyQv0M3kuY/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-1MVUGWXy5D0/T-GlfdEgggI/AAAAAAAAAYw/S_ebiSuuo2U/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

For some reason, the Fakes framework wasn’t generating fakes for the v 11 assemblies correctly. I eventually created a new Test project in VS 2012 RC, copied across the code and re-added references and fakes, and voila, everything is working great!

I’m not sure if this is a bug or not, so if you’re getting this error, my suggestion is to just create a new Test project in the RC and copy the code across.

Now back to (fake) testing…

