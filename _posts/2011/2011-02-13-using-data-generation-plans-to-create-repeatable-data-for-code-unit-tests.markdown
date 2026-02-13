---
layout: post
title: Using Data Generation Plans to create repeatable data for code unit tests
date: '2011-02-13 03:25:00'
tags:
- build
---

The Database Professional tools in VS 2010 allow you to create a project that encapsulates the schema of a SQL database. You can also use this project to create unit tests for stored procs or functions – and create test data for these database unit tests.

However, what if just want repeatable data for normal code unit tests as part of a TeamBuild? I was recently at a client where this scenario came up. I tried to get the data generation plan to run in the [AssemblyInitialize] method of the unit test project, but just got the error “Could not deploy database” or “Could not run data generation plan” – not very helpful.

So we figured out a workaround – essentially we use MSBuild to run the data generation plan in the solution just before unit testing.

Here are the high-level steps you need to follow:

- Create a DBPro project with a data generation plan
- Create an MSBuild project file that can run the data generation plan
- Customize your build template
- Add 2 arguments – RunDataGeneration and PathToMSBuildProjFile
- Add a couple of build activities to invoke MSBuild to run the data generation plan

## The MSBuild Project File

Once you’ve created the database project (I won’t cover it in this post, but there are plenty of blogs and articles on the web about how to do this), create a data generation plan for your test data. Now create a new xml file in the root of your database project – the one I created is called datagen.proj. Here’s the file contents:

    <span style="color: blue">&lt;</span><span style="color: #a31515">Project </span><span style="color: red">DefaultTargets</span><span style="color: blue">=</span>"<span style="color: blue">DataGen</span>" <span style="color: red">xmlns</span><span style="color: blue">=</span>"<span style="color: blue"><a href="http://schemas.microsoft.com/developer/msbuild/2003">"&gt;http://schemas.microsoft.com/developer/msbuild/2003</a></span><a href="http://schemas.microsoft.com/developer/msbuild/2003">"<span style="color: blue">&gt;<br></span></a><br>&lt;<span style="color: #a31515">Import </span><span style="color: red">Project</span><span style="color: blue">=</span>"<span style="color: blue">$(MSBuildExtensionsPath)\Microsoft\VisualStudio\v10.0\TeamData\Microsoft.Data.Schema.Common.targets</span>" <span style="color: blue">/&gt; <br><br>&lt;</span><span style="color: #a31515">Target </span><span style="color: red">Name</span><span style="color: blue">=</span>"<span style="color: blue">DataGen</span>"<span style="color: blue">&gt;<br><br>&lt;</span><span style="color: #a31515">DataGeneratorTask<br><br></span><span style="color: red">ConnectionString</span><span style="color: blue">=</span>"<span style="color: blue">Data Source=dataserver;Initial Catalog=AutomatedTesting;Integrated Security=True;Pooling=False</span>"<br><br><span style="color: red">SourceFile</span><span style="color: blue">=</span>"<span style="color: blue">.\Data Generation Plans\UnitTestPlan.dgen</span>"<br><br><span style="color: red">PurgeTablesBeforePopulate</span><span style="color: blue">=</span>"<span style="color: blue">True</span>"<span style="color: blue">/&gt;<br><br><!--</span--><span style="color: #a31515">Target</span><span style="color: blue">&gt;<br><br><!--</span--><span style="color: #a31515">Project</span><span style="color: blue">&gt;</span></span></span>

Of course, you’ll need to change the connection string appropriately and set the “PurgeTablesBeforePopulate” to false if you don’t want to blow away the existing data – though since the data generation is creating test data, you should blow away existing data anyway. Set the sourcefile to the dgen file – the path is relative to the root of the database project.  
  
You can test the project file by opening a VS command console, navigating to the folder containing the proj file and typing “msbuild datagen.proj”. Make sure this step succeeds before you continue.

## Customizing the Build Workflow

I added two arguments to the workflow – a Boolean called “RunDataGeneration” and a string called DataGenMSBuildProjSourcePath. I also added metadata to “prettify” the arguments when builds are created.  
  
Now it’s time to customize the build workflow to create the test data before unit testing. I used the DefaultTemplate.xaml and navigated into the “Try, Compile and Test” sequence – then went deeper and found the “If Not TestSpecs = Emtpy” activity. A little further, just before the tasks that actually run the tests, I inserted an “If” activity (setting the condition to “RunDataGeneration = True”). The “else” branch is empty and I show the “Then” branch below:

<!--kg-card-begin: html-->[![screen](http://lh6.ggpht.com/_d41Ixos7YsM/TVbCmk_pTTI/AAAAAAAAAOc/UXQvfo8knHg/screen_thumb%5B6%5D.jpg?imgmax=800 "screen")](http://lh6.ggpht.com/_d41Ixos7YsM/TVbCk090_4I/AAAAAAAAAOY/og8wjpkQzuc/s1600-h/screen%5B4%5D.jpg)<!--kg-card-end: html-->

There are just two activities – a ConvertWorkSpace and an MSBuild activity. The ConvertWorkspace task is set to ServerToLocal and converts the DataGenMSBuildProjSourcePath to a local path (set the workspace parameter to “Workspace”). You’ll need to create a local variable scoped to the sequence called “localProjectPath” for the out parameter.  
  
Next, set the MSBuild task’s project to localProjectPath. The only other thing that needs mentioning here is that you MUST set the ToolPlatform of the MSBuild activity to X86 – the data generation seems to only work if called from the x86 msbuild and not from the x64 one.  
  
Check in your template and create the build – make sure you set “RunDataGeneration” to true and set the DataGenMSBuildProjSourcePath to the full source control path to the proj file (starting with $/…).  
  
Happy testing!

