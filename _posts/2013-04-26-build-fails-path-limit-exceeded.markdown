---
layout: post
title: 'Build Fails: Path Limit Exceeded'
date: '2013-04-26 21:54:00'
tags:
- build
---

I had a customer who mailed me about their builds failing. The error message was

<!--kg-card-begin: html--><font face="Courier New">Exception Message: The specified path, file name, or both are too long. The fully qualified file name must be less than 260 characters, and the directory name must be less than 248 characters.</font><!--kg-card-end: html--><!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-ehJSuaZ-iZ8/UXp49GeDtiI/AAAAAAAAAtQ/lhKBlndIVYM/image_thumb%25255B2%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-eQNhGS0pzL4/UXp47qTp-EI/AAAAAAAAAtI/luBrKVAZSMo/s1600-h/image%25255B4%25255D.png)<!--kg-card-end: html-->

The problem was the path of the source files got too long. There are 2 contributing factors for path length on a build server:

1. The Build Agent Working Directory setting
2. The Source Setting Workspace mapping

When the build agent checks out code, it checks it out to (WorkingDirectory)\Mapping for each mapping. By default, the build agent working directory is set to $(SystemDrive)\Builds\$(BuildAgentId)\$(BuildDefinitionPath) – more on these macros later – but this usually defaults the working directory that the source gets checked out to something like

<!--kg-card-begin: html--><font face="Courier New">c:\Builds\1\FabrikamFiber\FabrikamFiber.CallCenter MAIN\src</font><!--kg-card-end: html-->

(where the team project name is “FabrikamFiber” and the build definition name is “FabrikamFiber.CallCenter MAIN”).

Then, if you look at the Source Setting workspace mapping for the build, you’ll see the mappings for what source code the build is supposed to check out:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-8ZM6lMwJi2Y/UXp4_Y0GzVI/AAAAAAAAAtg/ujjdwXaCruU/image_thumb%25255B4%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-5MotCXdpb6g/UXp4-MNAX4I/AAAAAAAAAtY/c2sP7n2_cQw/s1600-h/image%25255B8%25255D.png)<!--kg-card-end: html-->

Here, $(SourceDir) equates to the root working folder for the build agent that ends up running the build. You’ll see I have a 2 mapping here – one to $(SourceDir) and one to $(SourceDir)\BuildLibs. That means that my BuildLibs will get checked out to:

<!--kg-card-begin: html--><font face="Courier New">c:\Builds\1\FabrikamFiber\FabrikamFiber.CallCenter MAIN\src\BuildLibs</font><!--kg-card-end: html-->

which is already 69 characters. If I have lots of subdirectories below that, I could start hitting the 260 character path limit.

## Customizing the Build Agent Working Directory

So let’s shorten the build agent working directory. In Team Explorer, click on the Build hub. Click “Actions” and select “Manage Build Controllers”.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-vUhsirYDlK8/UXp5B9eIajI/AAAAAAAAAtw/QGDK6l2oXRM/image_thumb%25255B6%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-mnyCUBYQQmE/UXp5Ag7OM7I/AAAAAAAAAto/iIZgqj8fbfY/s1600-h/image%25255B12%25255D.png)<!--kg-card-end: html-->

Then find the build agent(s) that are going to build your code and click “Properties”. You want to set the working directory to something shorter:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-PD04i9o8hIE/UXp5FAq2BwI/AAAAAAAAAuA/Tat0v1tXPLw/image_thumb%25255B8%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-XWq18c09NYQ/UXp5DsVVZ9I/AAAAAAAAAt4/GvX_JTOrijo/s1600-h/image%25255B16%25255D.png)<!--kg-card-end: html-->

There are 4 macros you can use here (according to [this page](http://msdn.microsoft.com/en-us/library/bb399135(v=vs.100).aspx)):

- 

**$(BuildAgentId)**: An automatically generated integer that uniquely identifies a build agent within a team project collection.

- 

**$(BuildAgentName)**: The Display Name of the build agent.

- 

**$(BuildDefinitionId)**: An automatically generated integer that uniquely identifies a build definition within a team project collection.

- **$(BuildDefinitionPath)**: The team project name and the build definition name, separated by a backslash.

So let’s change the Working Directory to:

<!--kg-card-begin: html--><font face="Courier New">c:\b\$(BuildAgentId)\$(BuildDefinitionId)</font><!--kg-card-end: html-->

That means the root of the working directory will be something like:

<!--kg-card-begin: html--><font face="Courier New">c:\b\1\10\src</font><!--kg-card-end: html-->

which is only 13 characters.

## Shorten the Build Source Setting Workspace Mapping

Nothing says that the mapped folder for the build has to have the same name as the folder in source control. For example, if you have

<!--kg-card-begin: html--><font face="Courier New">$/FabrikamFiber/Code In Some Really/Long Folder/That has SubFolders/Within SubFolders</font><!--kg-card-end: html-->

and you need to map a path to

<!--kg-card-begin: html--><font face="Courier New">$/FabrikamFiber/Code In Some Really/Long Folder/Libs</font><!--kg-card-end: html-->

for referencing a library, then you could map those 2 folders to

<!--kg-card-begin: html--><font face="Courier New">$(SourceDir)/T/W</font><!--kg-card-end: html-->

and

<!--kg-card-begin: html--><font face="Courier New">$(SourceDir)/Libs</font><!--kg-card-end: html-->

respectively. As long as you keep the same relative path “distances”, any relative path references you have will just work (assuming you haven’t hard coded them).

Happy building!

