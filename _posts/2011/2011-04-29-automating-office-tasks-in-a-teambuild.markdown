---
layout: post
title: Automating Office Tasks in a TeamBuild
date: '2011-04-29 21:01:00'
tags:
- build
---

I was working on a TeamBuild that doesn’t compile code – this build checks out a number of MS Word documents from source control, converts them to PDF and then uses [PDFSharp](http://pdfsharp.com/PDFsharp/) to merge them all into one PDF document. I started with the DefaultTemplate.xaml and ripped out the compile / test tasks. I then created a Powershell script that would do the heavy lifting. So all the build does is check out the doc files in the workspace, invoke the powershell script and then copy the final file to the drop folder. Seems pretty simple, right?

Well, in theory it was. I modified the workflow and created the Powershell script. I tested the Powershell script “manually” by calling it from a command prompt. I then got the workflow to call the script, expecting goodness. However, that’s where Simple ended and Frustration started – the script wouldn’t work when invoked from the workflow.

I tried to do several things to figure out the problem. Here’s the snippet of the code that was not working (the line in red):

<!--kg-card-begin: html--><font size="4" face="Cordia New">$word = new-object -ComObject "word.application"<br>    </font><!--kg-card-end: html--><!--kg-card-begin: html--><font size="4" face="Cordia New"><br>    $missing = [System.Reflection.Missing]::Value<br>   <font color="#008000"> # open read only<br></font>    <font color="#ff0000">$doc = $word.documents.open($source, $missing, $true)<br></font>    if ($doc -eq $null) {<br>        Throw "Could not open $source"<br>    }</font><!--kg-card-end: html-->

Whenever the script was invoked from within the build, the open() method to open the Word doc would return null and the script would Throw. For some reason, it couldn’t open the file. At first I thought it was an “interactive / non-interactive” problem, so I set the Visible property on the $word ComObject to false. No luck. I checked permissions. I checked the readonly attribute on the Word doc. Nothing helped.

## The Solution – the SystemProfile Desktop directory

I eventually stumbled onto some websites talking about a similar problem when doing Office automation from an ASP.NET site. They did all sorts of things with the identities and impersonation and so on, but that wouldn’t apply here since there’s no ASP.NET site.

There was one other piece of advice that turned out to solve the problem: the Desktop folder for the system profile. I was incredulous at first, but then decided what the heck and tried it.

All I had to do was create a folder called “Desktop” in the systemProfile folder (which is different for 64 bit and 32 bit machines). Once that folder existed, there was no more problem and the build worked like a charm. So I added in a sequence to check that the folder exists and to create it if it doesn’t. Here’s the activity:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/_d41Ixos7YsM/TbqofT2NXuI/AAAAAAAAAQg/lhXIymMmNaU/image_thumb%5B1%5D.png?imgmax=800 "image")](http://lh5.ggpht.com/_d41Ixos7YsM/Tbqod75P4sI/AAAAAAAAAQc/L1zVevQRBbY/s1600-h/image%5B3%5D.png)<!--kg-card-end: html-->

I added an If with the condition set to:

Directory.Exists(“C:\Windows\SysWOW64\config”)

In the “Then”, I assign “C:\Windows\SysWOW64\config\systemProfile\Desktop” to a variable called profileFolder, and in the Else I assign the profileFolder the value “C:\Windows\System32\config\systemProfile\Desktop”.

Next I added a “CreateDirectory” activity and pass in profileFolder as the directory to create.

I placed this sequence before any activity that performs Office automation and voila, the build works.

Happy building!

