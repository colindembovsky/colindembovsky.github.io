---
layout: post
title: 'Release Management 2015 with Build vNext: Component to Artifact Name Matching
  and Other Fun Gotchas'
date: '2015-07-29 15:55:14'
tags:
- releasemanagement
- build
---

I’ve been setting up a couple of VMs in Azure with a TFS demo. Part of the demo is release management, and I finally got to upgrade Release Management to the 2015 release. I wanted to test integrating with the new build vNext engine. I faced some “fun” gotchas along the way. Here are my findings.

## Fail: 0 artifacts(s) found

After upgrading Release Management server, I gleefully associated a component with my build vNext build. I was happy when the build vNext builds appeared in the drop-down. Since I was using the root of the output folder, I simply selected “\” as the location of the component (since I have several folders that I want to use via scripts, so I usually just specify the root of the drop folder).

I then queued the release – and the deployment failed almost instantly. “0 artifact(s) found corresponding to the name ‘FabFiber’ for BuildId: 91”. After a bit of head-scratching and head-to-table-banging, I wondered if the error was hinting at the fact that RM is actually looking for a published artifact named “FabFiber” in my build. Turns out that was correct.

## Component Names and Artifact Names

To make a long story short: you have to match the component name in Release Management with the artifact name in your build vNext “Publish Artifact” task. This may seem like a good idea, but for me it’s a pain, since I usually split my artifacts into Scripts, Sites, DBs etc. and publish each as a separate artifact so that I get a neat folder layout for my artifacts. Since I use PowerShell scripts to deploy, I used to specify the root folder “\” for the component location and then used Scripts\someScript.ps1 as the path to the script. So I had to go back to my build and add a PowerShell script to first put all the folders into a “root” folder for me and then use a single “Publish Artifacts” task to publish the neatly laid out folder structure. I looked at [this post](http://www.codewrecks.com/blog/index.php/2015/06/30/manage-artifacts-with-tfs-build-vnext/) from my friend Ricci Gian Maria to get some inspiration!

Here’s the script that I created and checked into source control:

    param(
        [string]$srcDir,
        [string]$targetDir,
        [string]$fileFilter = "*.*"
    )
    
    if (-not (Test-Path $targetDir)) {
        Write-Host "Creating $targetDir"
        mkdir $targetDir
    }
    
    Write-Host "Executing xcopy /y '$srcDir\$fileFilter' $targetDir"
    xcopy /y "$srcDir\$fileFilter" $targetDir
    
    Write-Host "Done!"

Now I have a couple of PowerShell tasks that copy the binaries (and other files) in the staging directory – which I am using as the root folder for my artifacts. I configure the msbuild arguments to publish the website webdeploy package to $(build.stagingDirectory)\FabFiber, so I don’t need to copy it, since it’s already in the staging folder. For the DB components and scripts:

- I configure the copy scripts to copy my DB components (dacpacs and publish.xmls) so I need 2 scripts which have the following args respectively:
- -srcDir MAIN\FabFiber\FabrikamFiber.CallCenter\FabrikamFiber.Schema\bin\$(BuildConfiguration) -targetDir $(build.stagingDirectory)\db -fileFilter \*.dacpac
- -srcDir MAIN\FabFiber\FabrikamFiber.CallCenter\FabrikamFiber.Schema\bin\$(BuildConfiguration) -targetDir $(build.stagingDirectory)\DB -fileFilter \*.publish.xml
- I copy the scripts folder directly from the workspace into the staging folder using these arguments:
- -srcDir MAIN\FabFiber\DscScripts -targetDir $(build.stagingDirectory)\Scripts
- Finally, I publish the artifacts like so:
<!--kg-card-begin: html-->[![image](/assets/images/files/195bdd4c-4072-4709-9fcd-a0a63af64d70.png "image")](/assets/images/files/a3119a20-6a71-4853-8ddc-641d544411fb.png)<!--kg-card-end: html-->

Now my build artifact (yes, a single artifact) looks as follows:

<!--kg-card-begin: html-->[![image](/assets/images/files/5ad836ef-244d-46d5-a9fa-465161d97093.png "image")](/assets/images/files/3f6230c3-b086-4ad6-be08-7637a600b25a.png)<!--kg-card-end: html-->

Back in Release Management, I made sure I had a component named “FabFiber” (to match the name of the artifact from the Publish Artifact task). I then also supplied “\FabFiber” as the root folder for my components:

<!--kg-card-begin: html-->[![image](/assets/images/files/c7cb6d5f-e997-4899-abad-a8ce8343c067.png "image")](/assets/images/files/d851e635-0537-49d6-98a5-f3c02d09db99.png)<!--kg-card-end: html-->

That at least cleared up the “cannot find artifact” error.

A bonus of this is that you can now use server drops for releases instead of having to use shared folder drops. Just remember that if you choose to do this, you have to set up a ReleaseManagementShare folder. See [this post](http://blogs.msdn.com/b/visualstudioalm/archive/2014/11/11/what-s-new-in-release-management-for-vs-2013-update-4.aspx) for more details (see point 7). I couldn’t get this to work for some reason so I reverted to a shared folder drop on the build.

## Renaming Components Gotcha

During my experimentation I renamed the component in Release Management that I was using in the release. This caused some strange behavior when trying to create releases: the build version picker was missing:

<!--kg-card-begin: html-->[![image](/assets/images/files/3e2dd5b6-b22d-44e3-b785-ebfb16aaa240.png "image")](/assets/images/files/72187323-5516-4637-b0de-4789ae8c22b5.png)<!--kg-card-end: html-->

I had to open the release template and set the component from the drop-down everywhere that it was referenced!

<!--kg-card-begin: html--> [![image](/assets/images/files/7ff7d1ae-0492-4ece-a747-7d2e9882e902.png "image")](/assets/images/files/f080521b-ab6f-4dff-85f5-478cd4c77b96.png)<!--kg-card-end: html-->

Once that was done, I got the build version picker back:

<!--kg-card-begin: html-->[![image](/assets/images/files/f515207a-6b1d-455e-b53b-e368e47e3853.png "image")](/assets/images/files/f1d0cb99-2307-4847-bc0a-b6485e2ce1a6.png)<!--kg-card-end: html-->

Deployments started working again – [my name is Boris, and I am invincible](https://www.youtube.com/watch?v=b18DjXWyWuc)!

<!--kg-card-begin: html--> [![image](/assets/images/files/98897692-3bd7-4f88-85ce-e311e02ad465.png "image")](/assets/images/files/8c7af39b-f730-4585-ba6b-fb41ca58befb.png)<!--kg-card-end: html-->
## The Parameter is Incorrect

A further error I encountered had to do with the upgrade from RM 2013. At least, I think that was the cause. The deployment would copy the files to the target server, but when the PowerShell task was invoked, I got a failure stating (helpfully – not!), “The parameter is incorrect.”

<!--kg-card-begin: html-->[![image](/assets/images/files/50078e51-ce0c-4f62-8a3e-78cd05e09024.png "image")](/assets/images/files/30296ba5-af98-49bd-9618-cd50fd3fe04a.png)<!--kg-card-end: html-->

At first I thought it was an error in my script – turns out that all you have to do to resolve this one is re-enter the password in the credentials for the PowerShell task in the release template. All of them. Again. Sigh… Hopefully this is just me and doesn’t happen to you when you upgrade your RM server.

## Conclusion

I have to admit that I have a love-hate relationship with Release Management. It’s fast becoming more of a HATE-love relationship though. The single feature I see that it brings to the table is the approval workflow – the client is slow, the workflows are clunky and debugging is a pain.

I really can’t wait for the release of Web-based Release Management that will use the same engine as the build vNext engine, which should mean a vastly simpler authoring experience! Also the reporting and charting features we should see around releases are going to be great.

For now, the best advice I can give you regarding Release Management is to make sure you invest in agent-less deployments using PowerShell scripts. That way your upgrade path to the new Web-based Release Management will be much smoother and you’ll be able to reuse your investments (i.e. your scripts).

Perhaps your upgrade experiences will be happier than mine – I can only hope, dear reader, I can only hope.

Happy releasing!

