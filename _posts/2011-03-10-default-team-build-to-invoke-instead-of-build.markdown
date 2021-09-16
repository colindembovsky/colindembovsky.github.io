---
layout: post
title: Default Team Build to Invoke Instead of Build
date: '2011-03-10 05:37:00'
tags:
- build
---

Sometimes you need to do a “deployment” that doesn’t involve a build of source code – for example, I have been working at a customer in gorgeous Durban and they’ve got some Dynamix AX scripts that they need to source control – no problem for TFS. The “deployment” involves simply copying the entire source folder to a secure share.

So I sat down with Dave Pike, one of their senior developers / architects and we customized the default team build template. Essentially, we ripped out the MSBuild task that compiles, the testing tasks and some of the arguments that you specify on a default build. We still wanted the build to create the workspace, check out some folder (or folders), label source for the build and associate changesets and work items. Where the MSBuild task used to be, we wanted to invoke a script.

Finally, we changed the “copy binaries to drop folder” task to copy the sources to the drop location – this gives you the “build output” in the drop location.

## Source Control Structure

In our case, we wanted to check out a folder from source control and invoke a command within that folder to do an xcopy. Here’s how we structured the folders:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/_d41Ixos7YsM/TXfWxcad5UI/AAAAAAAAAPg/hZpTSlhKjOk/image_thumb%5B2%5D.png?imgmax=800 "image")](http://lh5.ggpht.com/_d41Ixos7YsM/TXfWvLzhc2I/AAAAAAAAAPc/P3t0dd1hZUM/s1600-h/image%5B4%5D.png)<!--kg-card-end: html-->

The tree in source control that we set as the workspace for the build (and hence the root SourcesDirectory on the build agent when the build is running) was $/TeamProject/DEV/Scripts. Within that folder, we had some other folders and the Deploy.cmd. When setting up the build definition, we specified the “Source Control Path to Script” as $/TeamProject/DEV/Scripts/Deploy.cmd – the build was able to invoke the script from there. The script is executed using the Sources directory (the root of the workspace mapping on the build agent) so you can work in relative paths from there if you’re copying or manipulating files in the workspace. In this case the working folder would be the local mapping of $/TeamProject/DEV/Scripts.

## The Deploy.cmd File

You’ll have to create a script (a Powershell script or a bat file or any other script you can invoke) that does the “deployment”. In our case, we wanted to xcopy the entire sources directory (the entire workspace that we set up when we create a build) to a UNC, so we just created a cmd file. We added this to the source control folder that held the scripts we wanted to deploy so that it would be checked out when the build runs. The source control path to this script is the only argument that you really need to specify when you set up one of these builds.

For example,

Here’s a stub for a generic deploy.cmd file that reports errors back to the build (so that the whole build is failed if the script fails):

    <font color="#0000ff">@echo off<br> <br><font color="#ff0000">rem =============<br>rem do stuff here<br>rem =============</font><br> <br>IF %ERRORLEVEL% NEQ 0 GOTO Err<br>GOTO End<br> <br>:Err<br>@echo An error occurred<br>exit /B 1<br> <br>:End<br>@echo Done!</font>

    <font face="Arial">Obviously you’ll replace the actual “work” that you want this script to perform where the red “rems” are. That was where we put our xcopy.</font>

## Gotchas For Associating Changesets and Work Items

When we had dropped in an Invoke build activity, we ran the build. The builds worked fine, but weren’t calculating the changesets or work items associated to this build. We opened the template again and for a while we were stumped – we had the AssociateChangeSetsAndWorkItems activity there and hadn’t messed with it at all – so why were we not seeing the associated changesets and work items?

We examined the build log of a few of the builds and realised that the AssociateChangeSetsAndWorkItems activity was logging a warning that said, “Cannot find label ‘’”. Reflecting the task revealed that the activity calculates that “last good label” for this build and then using dates, works out all the changesets and associated work items from the date of the last good build label to now and associates them with this current build. We could see that the logs were labelling the builds and we could see the labels in source control, but still the activity couldn’t seem to work out the “last good build label”. Then we realised that in the sequence of the Default Template where the MSBuild activity is executed, the compilation and test statuses are set. We’d ripped out that sequence when we added the Invoke, so we weren’t setting the compilation or test statuses at all. We simply added a SetBuildDetail activity just after our Invoke activity and set compilation, test and overall status on the BuildDetail to Successful – and voila, the builds were now able to work out the changesets and work items since the “last good build” and associate them with this build.

I suppose that makes sense – TFS only sets the build as “good” if the compilation and test status are both successful – we set that after our Invoke (if the Invoke fails because of error in the script, the status is not set to Success and the build is not classified as “good”).

## The Workflow

Here’s the Workflow “summary”:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/_d41Ixos7YsM/TXfW04xoDzI/AAAAAAAAAPo/CqKjFUnBKxY/image_thumb%5B20%5D.png?imgmax=800 "image")](http://lh6.ggpht.com/_d41Ixos7YsM/TXfWy2zHuCI/AAAAAAAAAPk/qyW232vVSEw/s1600-h/image%5B14%5D.png)<!--kg-card-end: html-->

This is the detail of the parallel activity inside the “Try Invoke and Associate Changesets and Work Items:

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/_d41Ixos7YsM/TXfW41xltkI/AAAAAAAAAPw/9p39E25T3Yk/image_thumb%5B26%5D.png?imgmax=800 "image")](http://lh3.ggpht.com/_d41Ixos7YsM/TXfW2l-JvnI/AAAAAAAAAPs/2mX3fsCxxaw/s1600-h/image%5B19%5D.png)<!--kg-card-end: html-->

Finally, here’s the sequence that does the deployment:

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/_d41Ixos7YsM/TXfW9Uob7TI/AAAAAAAAAP4/OFI43AxR0Sg/image_thumb%5B34%5D.png?imgmax=800 "image")](http://lh4.ggpht.com/_d41Ixos7YsM/TXfW6a6fgDI/AAAAAAAAAP0/pVoCtOfyeuU/s1600-h/image%5B26%5D.png)<!--kg-card-end: html-->

You can get the template from my [skydrive](http://cid-64a24e0938d6d062.office.live.com/self.aspx/Colin%5E4s%20ALM%20Corner/NoCompileInvokeScriptTemplate.xaml).

Happy building!

