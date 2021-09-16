---
layout: post
title: Managing Config for .NET Core Web App Deployments with Tokenizer and ReplaceTokens
  Tasks
date: '2016-10-26 10:20:20'
tags:
- releasemanagement
- build
---

Last week I posted an [end-to-end walkthrough](/end-to-end-walkthrough-deploying-web-applications-using-team-build-and-release-management) about how to build and deploy web apps using Team Build and Release Management – including config management. The post certainly helps you if you’re on the .NET 4.x Framework – but what about deploying .NET Core apps?

## The Build Once Principle

If you’ve ever read any of my blogs you’ll know I’m a proponent of the “build once” principle. That is, your build should be taking source code and (after testing and code analysis etc.) producing a _single package_ that can be deployed to multiple environments. The biggest challenge with a “build once” approach is that it’s non-trivial to manage configuration. If you’re building a single package, how do you deploy it to multiple environments when the configuration is different on those environments? I present a solution in my walkthrough – use a publish profile and a parameters.xml file to tokenize the configuration file during build. Then replace the tokens with environment values at deploy time. I show you how to do that starting with the required source changes, how the build works and finally how to craft your release definition for token replacements and deployment.

## AppSettings.json

However, .NET Core apps are a different kettle of fish. There is no web.config file (by default). If you File-\>New Project and create a .NET Core web app, you’ll get an appsettings.json file. This is the “new” web.config if you will. If you then go to the .NET Core documentation, you’ll see that you can create multiple configuration files using “magic” names like appsettings.dev.json and appsettings.prod.json (these are loaded up during Startup.cs). I understand the appeal of this approach, but to me it feels like having multiple web.config files which you replace at deployment time (like web.dev.config and web.prod.config). I’m not even talking about config transforms – just full config files that you keep in source control and (conceptually) overwrite during deployment. So you’re duplicating code – which is bad juju.

I got to thinking about how to handle configuration for .NET Core apps, and after mulling it over and having a good breakfast chat fellow MVP [Scott Addie](https://scottaddie.com/), I thought about tokenizing the appsettings.json file. If I could figure out a clean way to tokenize the file at build time, then I could use my existing [ReplaceTokens](https://github.com/colindembovsky/cols-agent-tasks/tree/master/Tasks/ReplaceTokens) task (part of my [marketplace extension](https://marketplace.visualstudio.com/items?itemName=colinsalmcorner.colinsalmcorner-buildtasks)) during deploy time to fill in environment specific values. Unfortunately there’s no config transform for JSON files, so I decided to create a [Tokenizer](https://github.com/colindembovsky/cols-agent-tasks/tree/master/Tasks/Tokenizer) task that could read in a JSON file and then auto-replace values with tokens (based on the object hierarchy).

## Tokenizer Task

To see this in action, I created a new .NET Core Web App in Visual Studio. I then added a custom config section. I ended up with an appsettings.json file that looks as follows:

    {
      "ConnectionStrings": {
        "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=aspnet-WebApplication1-26e8893e-d7c0-4fc6-8aab-29b59971d622;Trusted_Connection=True;MultipleActiveResultSets=true"
      },
      "Tricky": {
        "Gollum": "Smeagol",
        "Hobbit": "Frodo"
      },
      "Logging": {
        "IncludeScopes": false,
        "LogLevel": {
          "Default": "Debug",
          "System": "Information",
          "Microsoft": "Information"
        }
      }
    }

Looking at this config, I can see that I might want to change the ConnectionStrings.DefaultConnection as well as the Tricky.Gollum and Tricky.Hobbit settings (yes, I’m reading the Lord of the Rings – I’ve read it about once a year since I was 11). I may want to change Logging.LogLevel.Default too.

Since the file is JSON, I figured I could create a task that reads the file in and then walks the object hierarchy, replacing values with tokens as it goes. But I realized that you may not want to replace every value in the file, so the task would have to take an explicit include (for only replacing certain values) or exclude list (for replacing all but certain values).

I wanted the appsettings file to look like this once the tokenization had completed:

    {
      "ConnectionStrings": {
        "DefaultConnection": " __ConnectionStrings.DefaultConnection__"
      },
      "Tricky": {
        "Gollum": " __Tricky.Gollum__",
        "Hobbit": " __Tricky.Hobbit__"
      },
      "Logging": {
        "IncludeScopes": false,
        "LogLevel": {
          "Default": " __Logging.LogLevel.Default__",
          "System": "Information",
          "Microsoft": "Information"
        }
      }
    }

You can see the tokens on the highlighted lines.

After coding for a while on the plane (#RoadWarrior) I was able to create a task for tokenizing a JSON file (perhaps in the future I’ll make more file types available – or I’ll get some Pull Requests!). Having recently added unit tests for my Node tasks, I was able to bang this task out rather quickly.

## The Build Definition

With my shiny new Tokenize task, I was ready to see if I could get the app built and deployed. Here’s what my build definition looks like:

<!--kg-card-begin: html-->[![image](/assets/images/files/7b982804-8515-4c8d-a458-aa844cc2f3e5.png "image")](/assets/images/files/91122559-5470-46fa-b8f5-e9c464d5cb7f.png)<!--kg-card-end: html-->

The build tasks perform the following operations:

1. Run dotnet with argument “restore” (restores the package dependencies)
2. Tokenize the appsettings.json file
3. At this point I should have Test, Code Annalysis etc. – I’ve omitted these quality tasks for brevity
4. Run dotnet with arguments “publish src/CoreWebDeployMe --configuration $(BuildConfiguration) --output $(Build.ArtifactStagingDirectory)/Temp” (I’m publishing the folder that contains my .NET Core web app with the BuildConfiguration and placing the output in the Build.ArtifactStagingDirectory/Temp folder)
5. Zip the published folder (the zip task comes from [this extension](https://marketplace.visualstudio.com/items?itemName=Trackyon.trackyonadvantage))
6. Remove the temp folder from the staging directory (since all the files I need are now in the zip)
7. Upload the zip as a build drop

The Tokenize task is configured as follows:

<!--kg-card-begin: html-->[![image](/assets/images/files/5bfe206d-bff6-4adb-9dea-d89fb3c655e3.png "image")](/assets/images/files/a5a4f06d-e255-4521-ad9e-5a5b46666842.png)<!--kg-card-end: html-->

Let’s look at the arguments:

- Source Path – the path containing the file(s) I want to tokenize
- File Pattern – the mini-match pattern for the file(s) within the Source Path I want to tokenize
- Tokenize Type – I only have json for now
- IncludeFields – the list of properties in the json file that I want the Tokenizer to tokenize
- ExcludeFields – I could have used a list of properties I wanted to exclude from tokenization here instead of using the Include Fields property

Once the build completes, I now have a potentially deployable .NET Core web application with a tokenized appsettings file. I could have skipped the zip task and just uploaded the site unzipped, but uploading lots of little files takes longer than uploading a single larger file. Also, I was thinking about the deployment – downloading a single larger file (I guessed) was going to be faster than downloading a bunch of smaller files.

## The Release

I was expecting to have to unzip the zip file, replace the tokens in the appsettings.json file and then re-zip the file before invoking WebDeploy to push the zip file to Azure. However, the AzureRM WebDeploy task recently got updated, and I noticed that what used to be “Package File” was now “Package File or Folder”. So the release turned out to be really simple:

1. Unzip the zip file to a temp folder using an inline PowerShell script (why is there no complementary Unzip task from the Trackyon extension?)
2. Run ReplaceTokens on the appsettings.json file in the temp folder
3. Run AzureRM WebDeploy using the temp folder as the source folder
<!--kg-card-begin: html-->[![image](/assets/images/files/283a9350-102d-4f3f-b3ce-ea1c9a6722e9.png "image")](/assets/images/files/f39f1990-af9b-45c5-ac5c-30784ab19dab.png)<!--kg-card-end: html-->

Here’s how I configured the PowerShell task:

<!--kg-card-begin: html-->[![image](/assets/images/files/b241102d-fe0a-4e24-a4c6-3411e22cc4d4.png "image")](/assets/images/files/4b94b9a1-f720-4169-b63f-7fa3fdbf3e86.png)<!--kg-card-end: html-->

The script takes in the sourceFile (the zip file) as well as the target path (which I set to a temp folder in the drop folder):

    param(
      $sourceFile,
      $targetPath)
    
    Expand-Archive -Path $sourceFile -DestinationPath $targetPath -Force

My first attempt deployed the site – but the ReplaceTokens task didn’t replace any tokens. After digging a little I figured out why – the default regex pattern – \_\_(\w+)\_\_ – doesn’t work when the token name have periods in them. So I just updated the regex to \_\_(\w+[\.\w+]\*)\_\_ (which reads “find double underscore, followed by a word, followed by a period and word repeated 0 or more times, ending with double underscore”.

<!--kg-card-begin: html-->[![image](/assets/images/files/a17eccde-040d-422e-8254-5c21663d9785.png "image")](/assets/images/files/961fd32e-87b7-4a81-9d49-590d1e5833d5.png)<!--kg-card-end: html-->

That got me closer – one more change I had to make was replacing the period with underscore in the variable names on the environment:

<!--kg-card-begin: html-->[![image](/assets/images/files/1208a734-c236-472c-832c-6d2991e0f027.png "image")](/assets/images/files/be93b375-e5e0-4cde-ace9-41b6520e62ee.png)<!--kg-card-end: html-->

Once the ReplaceTokens task was working, the Deploy task was child’s play:

<!--kg-card-begin: html-->[![image](/assets/images/files/a5505b40-51b5-4c22-82bb-fd5706e59728.png "image")](/assets/images/files/9a92ca4f-f091-49cb-bbe0-8419047cbecc.png)<!--kg-card-end: html-->

I just made sure that the “Package or Folder” was set to the temp path where I unzipped the zip file in the first task. Of course at this point the appsettings.json now contains real environment-specific values instead of tokens, so WebDeploy can go and do its thing.

## Conclusion

It is possible to apply the Build Once principle to .NET Core web applications, with a little help from my friends Tokenizer and ReplaceTokens in the build and release respectively. I think this approach is fairly clean – you get to avoid duplication in source code, build a single package and deploy to multiple environments. Of course my experimentation is available to your for free from the tasks in my [marketplace extension](https://marketplace.visualstudio.com/items?itemName=colinsalmcorner.colinsalmcorner-buildtasks)! Sound off in the comments if you think this is useful (or horrible)…

Happy releasing!

