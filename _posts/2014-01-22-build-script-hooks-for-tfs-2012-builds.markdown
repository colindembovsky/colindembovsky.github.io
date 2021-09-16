---
layout: post
title: Build Script Hooks for TFS 2012 Builds
date: '2014-01-22 21:51:00'
tags:
- build
---

EDIT: My colleague Tyler Doerksen pointed out in his comments that my solution doesn’t do any error checking of the scripts. If your script fails, the build happily continues. I’ve added [another post](http://www.colinsalmcorner.com/2014/01/error-handling-poor-mans-runscript-in.html) to show how to add error handling.

One of my favorite features about the TFS 2013 Builds is the script hooks – there are pre- and post-build as well as pre- and post-test hooks. These make customizing build a whole lot easier. For example, customizing the build so that your assembly versions match your build number [is a snap](http://www.colinsalmcorner.com/2013/07/matching-binary-version-to-build-number.html).

I set out to implement the same logic in a 2012 build this morning – unfortunately, the RunScript build activity from the 2013 template is only in the 2013 TFS Build assembly.

So I came up with a “poor-man’s” run-script equivalent for 2012 builds (with the best part being you don’t need any custom assemblies, so the edit can be applied directly to your build template without having to pull it into a solution). I’ll walk you through the steps of customizing the default build – in this example I’m only doing pre- and post-build scripts, but the principles would be the same for pre- and post-test scripts.

## Challenge 1 – Invoking PowerShell

The first challenge is how do you invoke a PowerShell script from within the build? It’s fairly easy: use the InvokeProcess activity.

First you’ll need to add workflow arguments for the pre- and post-build script paths as well as their corresponding args. Open your build workflow and click on “Arguments”. Enter 4 “In String” arguments as follows:

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-n3XDr5QsH10/Ut-wpsUu8gI/AAAAAAAABLs/2mMlrsLAvYo/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-A0SlTFrXZK8/Ut-woxu6NEI/AAAAAAAABLk/LgyhQ3CP7zM/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

You can add defaults if you like.

I always like to put custom arguments in a separate section of the build parameters, so that anyone creating a build from the template can see them and read their descriptions. You do this by clicking the “…” button in the Default Value column of the “Metadata” argument and filling in some metadata for your arguments:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-UljBJ6TNYAQ/Ut-wq-Dg2CI/AAAAAAAABL8/ePeDUlft-2w/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-6Caj4I539nE/Ut-wqToYgPI/AAAAAAAABL0/d94g2x3w-G8/s1600-h/image%25255B7%25255D.png)<!--kg-card-end: html-->

Now go to the workflow and find the MSBuild Activity in the heart of the workflow that does the compilation (be careful – there’s one that does a clean of the workspace too – you don’t want that one). Just above the ForEach (For Each Project in BuildSettings.ProjectsToBuild) activity, add an If activity (this will automatically add in a sequence to wrap the activities we’re adding) and set its condition to

<!--kg-card-begin: html--><font size="2" face="Courier New">Not String.IsNullOrEmpty(PreBuildScriptPath)</font><!--kg-card-end: html-->

and in the Then of the If activity add a “ConvertWorkspaceItem” activity and an InvokeProcess activity. In the InvokeProcess activity, drag a “WriteBuildMessage” and “WriteBuildError” activity onto the area below stdOutput and errOutput respectively. Set the “Message” property of each Write activity to stdOutput and errOutput respectively.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-aS6oqjAyftg/Ut-wsXlgPgI/AAAAAAAABMM/IgNvSQMJLlY/image_thumb%25255B5%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-vIOv0L3w1IA/Ut-wrs-GLLI/AAAAAAAABME/7-8_QsbEE7M/s1600-h/image%25255B11%25255D.png)<!--kg-card-end: html-->

Click on the sequence activity that your activities are in. We’ll need two local variables: preBuildScriptLocalPath and postBuildScriptLocalPath (both strings).

Now in the ConvertWorkspaceItem activity, set the following properties:

- DisplayName: “Get pre-build script local path”
- Input: PreBuildScriptPath
- Output: preBuildScriptLocalPath
- Workspace: Workspace

Set the following properties in the InvokeProcess activity:

- Arguments: String.Format(" ""& '{0}' {1}"" ", prebuildScriptLocalPath, PreBuildScriptArgs)
- DisplayName: “Run pre-build script”
- FileName: “PowerShell”

Now you can copy this whole “If” activity and paste it below the “ForEach” (the one that does contains the MSBuild activity) and rename pre to post – this implements your post-build hook.

Don’t forget that you’ll need to change PowerShell’s execution policy on your build server. Log in to your build server and run PowerShell as an administrator. Run the following:

<!--kg-card-begin: html--><font size="2" face="Courier New">Set-ExecutionPolicy RemoteSigned</font><!--kg-card-end: html-->

Now you’re almost set…

## Challenge 2 – Environment Variables

When I created a script for a 2013 build to version the assemblies, I relied on the fact that the 2013 build sets some environment variables that you can use in your scripts. Here’s a snippet showing 2 environment variables I used in my [versioning script](http://www.colinsalmcorner.com/2013/07/matching-binary-version-to-build-number.html):

    Param(
      [string]$pathToSearch = $env:TF_BUILD_SOURCESDIRECTORY,
      [string]$buildNumber = $env:TF_BUILD_BUILDNUMBER,

You can see I’m getting $env:TF\_BUILD\_BUILDNUMBER. Well, in the 2012 workflow, these variables aren’t set, so you have to add an activity to do it.

Just above your “If” activity for the pre-build script invocation, add an InvokeMethod activity (this is in the “Primitives” section of the workflow designer toolbox).

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-EUWdOsDWxzI/Ut-wuAp2r1I/AAAAAAAABMc/i8yS-yCdVLA/image_thumb%25255B7%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-ZDfuvAp7Sy4/Ut-wtSBw6aI/AAAAAAAABMU/DptMqrg6gAc/s1600-h/image%25255B15%25255D.png)<!--kg-card-end: html-->

Set the following properties:

- MethodName: SetEnvironmentVariable
- TargetType: System.Environment

Then you need to set different parameters for each environment variable you want to set. Click on the “…” next to the value of the Parameters property:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-78axFRtXjIA/Ut-wvY49xeI/AAAAAAAABMs/p-6sSH_MTs8/image_thumb%25255B9%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-KvKAIkIe4AM/Ut-wusSu33I/AAAAAAAABMk/Cc82aqusQsU/s1600-h/image%25255B19%25255D.png)<!--kg-card-end: html-->

In this one I set 2 parameters: “In String TF\_BUILD\_SOURCESDIRECTORY” and “In String SourcesDirectory” to set the sources directory environment variable. I did the same for binaries directory and build number, each time using the name from [this list](http://msdn.microsoft.com/en-us/library/vstudio/dn376353.aspx#scripts) (see the TF\_BUILD environment variables section) and the value from the corresponding workflow argument or variable. Then I could use the same PowerShell script that I used in my 2013 builds without having to modify it.

## Creating a Build Definition

Now when you create a build definition, make sure that you include the folder that contains your scripts into the build workspace. Then set your script paths (using the source control paths) and arguments appropriately, for example:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-3V0EjbZKtsU/Ut-wwmH38OI/AAAAAAAABM8/6Ljc-YStxY4/image_thumb%25255B11%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-UacgKu522uo/Ut-wwHm4IiI/AAAAAAAABM0/mLHR81iOT5A/s1600-h/image%25255B23%25255D.png)<!--kg-card-end: html-->

Happy customizing!

